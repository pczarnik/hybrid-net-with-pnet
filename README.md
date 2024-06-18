# 1. Konfiguracja PNETLab

Maszyna wirtualna PNETLab wymaga WVMware Workstation, Player lub ESXi, ale poniższe laboratiorium było testowane wyłącznie na Workstation. Obraz PNETLab można pobrać ze strony https://pnetlab.com/pages/download.

1. Uruchom maszynę wirtualną <br>
Przed uruchomieniem maszyny, należy zaznaczyć w ustawieniach 'Virtualize Intel VT-x/EPT or AMD-V/RVI':
![Alt text](/images/Screenshot%20from%202024-06-18%2008-05-50.png)
Gdy maszyna się uruchomi, zanotuj przydzielony adres IP.

2. Po uruchomieniu maszyny wirtualnej, zaloguj się domyślnym loginem i hasłem (root/pnet). Jeśli uruchamiasz ją po raz pierwszy, odczekaj chwilę, aż pojawi się okno konfiguracji. Pozostaw wszędzie wartości domyślne, klikając 'Enter'.

3. Zainstaluj `ishare2`, za pomocą komendy:
```console
wget -O /usr/sbin/ishare2 https://raw.githubusercontent.com/ishare2-org/ishare2-cli/main/ishare2 && chmod +x /usr/sbin/ishare2 && ishare2
```

4. Pobierz wybrany obraz switcha. Możesz obejrzeć dostępne obrazy za pomocą komendy:
```console
ishare2 search viosl2
```
Aby pobrać jeden z nich wykonaj komendę:
```console
ishare2 pull <typ, np. qemu> <numer obrazu> 
```

5. Pobierz wybrany obraz routera. Możesz obejrzeć dostępne obrazy za pomocą komendy:
```console
ishare2 search csr
```
Aby pobrać jeden z nich wykonaj komendę:
```console
ishare2 pull <typ, np. qemu> <numer obrazu> 
```

6. Wpisz w przeglądarkę zanotowany wcześniej adres IP i zaloguj się do PNETLab w trybie offline. Następnie upewnij się, że możesz wybrać pobrane przez siebie router i switch. W tym celu kliknij prawym przyciskiem myszy i wybierz 'Node'. Powinieneś móć wybrać pobrane urządzenia.

7. Wyłącz maszynę writualną, a następnie odłącz host od internetu.

# 2. Topologia sieci

Zbuduj sieć jak na obrazku poniżej:


Chmurką zostało oznaczone 'Managment Cloud', które można dodać klikając prawym przyciskiem myszy i wybierając 'Network'.

# 3. Konfiguracja sieci

Każdemu routerowi (zarówno fizycznemu jak i wirtualnym) przypisz adres IP w tej samej sieci. Nadaj adres IP również hostom na których są wirtualne maszyny. Upewnij się, że routery są w stanie się komunikować.
<br>
Następnie skonfiguruj dostęp ssh. Poniższe kroki powtórz na każdym routerze, dla każdego wybierają unikalny hostname.

1. Ustaw hostname
```console
hostname h1
``` 
2. Ustaw domenę
```console
ip domain-name TIP.LOCAL
```

3. Wygeneruj klucz. Wybierz rozmiar klucza 2048.
```console
crypto key generate rsa
```

4. Włącz wyższą wersję ssh
```console
ip ssh version 2
```

5. Skonfiguruj VTY
```console
line vty 0 4
transport input ssh
login local
```

6. Utwórz użytkownika. Musi być to tama nazwa i hasło na wszystkich routerach!
```console
username admin password pass
```

7. Włacz możliwośc wykonania `enable` i `config t` przez ssh
```console
enable secret <hasło>
```

8. Ustaw wyższe uprawnienia użtykownikowi
```console
username admin privilege 15
```

# 4. Tworzenie loopbacków i OSPF za pomocą ansible

W pliku `inventory.cfg` pod `[all]` wpisz skonfigurowane routery zgodnie ze wzorem: `<hostname> ansible_host=<ip address>`. W pliku vars.yaml każdemu z routerów przydziel listę loopbacków. Następnie wykonaj polecenie:
```console
ansible-playbook -i inventory.cfg config_routers.yaml
```
Powinny wykonać się zadania, które najpierw usuną loopbacki i obecną konfigurację OSPF, utowrzą nowe interfejsy loopback, a następnie skonfigurują OSPF. Na koniec powinna być możliwość komunikacji między loopbackami na różnych routerach.

# 5. Możliwe problemy

1. Wersja 6.1.2 kolecji `ansible.netcommon` zawiera błędy, które nie pozwalją na tworzenie interfejsów. Należy zainstalować starszą wersję, np. 6.1.1.

2. Aby PNETLab działał prawidłowo musi mieć możliwość działania w trybie promiscous. W tym celu w systemie Ubuntu naleźy wykonać komendę: `chmod a+rw /dev/vmnet*` przed uruchomieniem maszyny wirtualnej.
