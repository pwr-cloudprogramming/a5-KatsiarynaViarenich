### A5. Terraform. Uruchomienia aplikacji na ec2.

Poniżej są wymienione pięć problemów, z którymi się spotkałam podczas wykonania tego zadania.

1. Dla mnie, jako dla osoby prawie nie posiadającej doświadczenia w frameworku spring, pierwszym wyzwaniem został mechanizm CORS. Po dużej ilości przeczytanego materiału i dużej liczbie prób i błędów, udało się pozwolić na komunikację pomiędzy frontendem a backendem kodu w następujący sposób:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Value("${ip_address}")
    private String ipAddress;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/game/**")
                .allowedOrigins("http://" + ipAddress + ":8081");
    }
}
```

2. Następnym problemem był problem z wersją javy. Moja aplikacja wymagała instalacji java 17, która była za duża i nie mogła się zmieścić na żadnej z instancji ec2 - ani na instancji utworzonej przez Cloud9, ani na instancji tworzonej przez terraform. Rozwiązałam ten problem, usuwając niepotrzebne rzeczy z pamięci Cloud9 i wybierając najmniej ważący obraz javy 17 (w tym przypadku to było amazoncorretto:17).

3. Następnym problemem został mój ustawiony "na sztywno" IP adres. Aby zrobić applikację bardziej "gibką", musiałam:

- Pobrać IP address do zmiennej środowiskowej
  
```bash
IP_ADDRESS=$(curl -s ifconfig.me)
```

- W przypadku backendu, zmodyfikować kod dodając application.properties wraz ze zmienną ipAddress - aby wprowadzać zmiany tylko do jednej zmiennej globalnej, a nie do pięciu róźnych linijek w całym kodzie.

- I w chmurze nadpisać tę zmienną ipAddress:

```bash
echo "ip_address=$IP_ADDRESS" > application.properties
```

- W przypadku frontendu, nadpisać pierwszą linijkę w kodzie socker_js.js:

```bash
sed -i "1s/.*/const url = 'http:\/\/$IP_ADDRESS:8080';/" socket_js.js
```

Podsumowując, w taki sposób, udało się pozbyć "sztywnego" adresu IP z całej aplikacji.

4. Przed napisaniem skryptu run.sh testowałam polecenia łącząc się do instancji ec2 poprzez ssh. Kiedy polecenia były przygotowane i byłam pewna, że moja konfiguracja jest poprawna, uruchomiłam terraform.

Ale moja aplikacja nie zadziałała. W log'ach nie udało się znaleźć żadnych błędów. Nawet nie byłam pewna, że polecenie "git clone" się uruchomiło, dlatego że nigdzie na dysku instancji nie mogłam zneleźć swojego repozytorium.

Ten problem udało się rozwiązać w bardzo prosty sposób, robiąc przed poleceniem "git clone" polecenie "cd /home/ubuntu". Po tej zmianie już mogłam widzieć swoje repozytorium i naprawiać konfigurację w przypadkach, jeżeli uruchominie dockera, nadpisanie ip adresu albo cokolwiek innego działało w niepoprawny sposób.

5. Nadużycie sudo i niepoprawna instalacja dockera.

Z tego powodu, że użytkownik nie posiadał uprawnień na urochomienie dockera, musiałam albo korzystać z instancji z poziomu użytkownika root, albo do wszystkich poleceń docker dodawać polecenie sudo. 
Aby naprawić tę sytuację, nadałam użytkownikowi ubuntu uprawnienia do urochomienie dockera:

```bash
usermod -aG docker ubuntu
```

Jednakże, to zadziałało nie od razu. Okazało się, że docker, zainstalowany za pomocą snap, nie tworzy swojej grupy użytkowników w sposób automatyczny. Wtedy, aby pozbyć się tego problemu, zainstalowałam docker pobierając go z get.docker.com:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```
Po takiej zmianie aplikacja się uruchomiła bez żadnych błędów.




