# Programowanie Full-Stack w Chmurze Obliczeniowej

      Marek Prokopiuk
      grupa dziekańska: 2.3
      numer albumu: 097710
## Laboratorium 7<br>Metody konfiguracji Service typu ClusterIP oraz NodePort. Wykorzystanie DNS w procesie wykrywania usług.
<p align="justify">Przedstawione zostało rozwiązanie zadania do laboratorium 7 dotyczącego użycia Network Plugins Calico, podziału i parametrów obiektów Service, metod konfiguracji Service typu ClusterIP oraz NodePort, a także wykorzystania DNS w procesie wykrywania usług. Ćwiczenie obejmowało konfigurację usług typu <b>ClusterIP</b> i <b>NodePort</b>, testowanie rozwiązań sieciowych za pomocą narzędzi dostępnych w kontenerach (m.in. <i>wget</i> oraz <i>nslookup</i>), a także weryfikację działania wbudowanego systemu DNS Kubernetes. </p>

---

### Utworzenie nowej przestrzeni nazw
<p align="justify">Pierwszym etapem zadania było przygotowanie dedykowanej przestrzeni nazw, w której miał zostać uruchomiony Pod wykorzystywany w dalszej części zadania. Utworzona została nowa przestrzeń nazw <i>remote</i>, a następnie zweryfikowano poprawność tej operacji, co potwierdziło, że przestrzeń została utworzona i jest aktywna. Wykorzystano w tym celu poniższe polecenia:</p>

```diff
kubectl create ns remote
kubectl get ns remote
```
<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Lab7/blob/main/screeny/1.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 1. Utworzenie przestrzeni nazw remote</i>
</p>

---

### Utworzenie Pod-a <i>remoteweb</i> oraz konfiguracja usługi typu NodePort
<p align="justify"> Kolejnym etapem było przygotowanie Pod-a działającego jako serwer WWW na bazie obrazu Nginx. W przestrzeni nazw <i>remote</i> utworzono Pod-a o nazwie <i>remoteweb</i>. Następnie udostępniono ten Pod jako usługę, tworząc obiekt Service typu <b>NodePort</b>. Wygenerowany losowo port NodePort został następnie zmodyfikowany do wartości wymaganej w zadaniu, czyli <i>31999</i>. Dokonano tego przy użyciu edytora polecenia, gdzie zmieniono pole <code>nodePort</code>. Finalnie zweryfikowano poprawną konfigurację ustawienia portu. Opisane czynności zrealizowane zostały z wykorzystaniem poniższych poleceń, a ich wyniki oraz wyedytowaną wartość <code>nodePort</code> widać na zrzutach ekranu.</p>

```diff
kubectl run remoteweb --image=nginx -n remote
kubectl expose pod remoteweb --type=NodePort --port=80 --target-port=80 -n remote
kubectl edit svc remoteweb -n remote
kubectl get svc -n remote
```
<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Lab7/blob/main/screeny/3.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 2. Utworzenie Pod-a oraz konfiguracja usługi typu NodePort</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Lab7/blob/main/screeny/2.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 3. Specyfikacja usługi po dokonanej zmianie</i>
</p>

---

### Uruchomienie Pod-a <i>testpod</i> w przestrzeni domyślnej
<p align="justify"> Aby możliwe było przetestowanie komunikacji między przestrzeniami nazw, uruchomiono dodatkowy Pod w przestrzeni domyślnej oparty na obrazie Busybox. Użyto polecenia</p>

```diff
kubectl run testpod --image=busybox -l app=testpod -- sleep infinity
```
  
<p align="justify">aby kontener pozostawał aktywny i umożliwiał wykonanie testów sieciowych.</p>
<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Lab7/blob/main/screeny/4.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 4. Uruchomienie Pod-a testpod w przestrzeni domyślnej</i>
</p>

---

### Weryfikacja dostępu do usługi z Pod-a testpod (DNS + wget)
<p align="justify"> Kolejnym krokiem było sprawdzenie, czy Pod w przestrzeni domyślnej może odnaleźć usługę uruchomioną w przestrzeni <i>remote</i>. W tym celu wykonano zapytanie DNS z testpod-a, korzystając z <i>nslookup</i> na pełnej nazwie domenowej:</p>

```diff
kubectl exec -it testpod -- nslookup remoteweb.remote.svc.cluster.local
```

<p align="justify">Wynik potwierdził, że DNS prawidłowo rozpoznaje usługę i zwraca jej adres IP typu ClusterIP. Następnie, używając polecenia</p>

```diff
kubectl exec -it testpod -- wget --spider --timeout=1 remoteweb.remote.svc.cluster.local
```

<p align="justify">, sprawdzono, czy z testpod-a możliwe jest połączenie z serwerem Nginx. Otrzymana odpowiedź „remote file exists” potwierdziła działanie usługi. </p>
<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Lab7/blob/main/screeny/5.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 5. Weryfikacja dostępu do usługi z Pod-a testpod</i>
</p>

---

### Weryfikacja dostępności usługi na porcie NodePort 31999
<p align="justify"> Ostatnią częścią zadania było sprawdzenie, czy serwer Nginx jest dostępny poprzez port NodePort ustawiony na <i>31999</i>. W środowisku Minikube działającym w WSL2, dostęp z zewnątrz możliwy jest poprzez funkcję tunelowania realizowaną przez polecenie <code>minikube service</code>. Uruchomienie polecenia</p> 

```diff
minikube service remoteweb --url -n remote
```

<p align="justify">spowodowało wygenerowanie lokalnego adresu typu <code>127.0.0.1:random_port</code>, który przekierowuje ruch do NodePort 31999 wewnątrz klastra. Następnie za pomocą <code>curl</code> pobrano stronę Nginx, co potwierdziło, że usługa działa i jest prawidłowo dostępna przez NodePort. Zachowanie, w którym Minikube udostępnia tunel na losowym porcie localhost, jest typowe dla konfiguracji Minikube z Docker driverem.</p>
<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Lab7/blob/main/screeny/6.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 6. Weryfikacja dostępności usługi poleceniem curl</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Lab7/blob/main/screeny/7.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 7. Weryfikacja dostępności usługi w przeglądarce</i>
</p>
