# 1. Konfiguracja PNETLab

Maszyna wirtualna PNETLab wymaga WVMware Workstation, Player lub ESXi, ale poniższe laboratiorium było testowane wyłącznie na Workstation. Obraz PNETLab można pobrać ze strony https://pnetlab.com/pages/download.
Pobrany plik będzie miał rozszerzenie `.ova`.

Do pierwszej konfiguracji PNETLaba konieczne jest posiadanie dostępu do publicznego internetu - najlepiej z adresem IP przydzielanym przez DHCP. 

1. Stwórz maszynę importując plik `.ova`.

   ![](/images/open_vm.png)

2. Zmodyfikuj ustawienia maszyny wirtualnej:
   - włącz zagnieżdżoną wirtualizację - `Virtualize Intel VT-x/EPT or AMD-V/RVI`
      ![](/images/enable_vtx.png)
   - ustaw interfejsy sieciowe (kolejność ma znaczenie!)
      ![](/images/net_if1.png)
      ![](/images/net_if2.png)

3. Uruchom maszynę wirtualną - zaloguj się za pomocą `root/pnet`
   ![](/images/vm_firstboot.png)

4. Pojawi się okno konfiguracji - nic nie zmieniaj, przeklikaj enterem.
   Maszyna automatycznie się zrestartuje.
   ![](/images/vm_firstboot_config.png)

5. Po automatycznym zrestartowaniu maszyny, zanotuj jej adres IP przydzielony przez DHCP.
   ![](/images/vm_ipaddress.png)
Jeśli adres nie zostanie przydzielony, zaloguj się do niej za pomocą `root/pnet` i ustaw statyczny adres IP:
   ```
   ip address add dev pnet0 <IP/MASK 192.168.0.5/24>
   ```

6. Sprawdź, czy maszyna jest dostępna pingując ją z hypervisora:
   ```
   ping <IP 192.168.0.5>
   ```

7. Teraz możesz zalogować się do PNETLaba przez ssh - wykorzystaj terminal lub PUTTY - login i hasło takie same jak wcześniej: `root/pnet`

8. Zainstaluj `ishare2`, za pomocą komendy:
   ```console
   wget -O /usr/sbin/ishare2 https://raw.githubusercontent.com/ishare2-org/ishare2-cli/main/ishare2 && chmod +x /usr/sbin/ishare2 && ishare2
   ```
   Przy automatycznej konfiguracji ishare2 skorzystaj z domyślnych ustawień zmieniając jedynie `Check SSL certrificate` na `n`.
   ![](/images/ishare_installation.png)

9. Pobierz obrazy routera i switcha:
   - sprawdź id-ki obrazów:
      ```
      ishare2 search vios-15.5.3M
      ishare2 search viosl2-15.2.4.55e
      ```
   - pobierz obrazy wykorzystując id-ki:
      ```
      ishare2 pull qemu 918
      ishare2 pull qemu 930
      ```

6. Wpisz w przeglądarkę zanotowany wcześniej adres IP i zaloguj się do PNETLab w trybie offline. Następnie upewnij się, że możesz wybrać pobrane przez siebie router i switch. W tym celu kliknij prawym przyciskiem myszy i wybierz 'Node'. Powinieneś móć wybrać pobrane urządzenia.

7. Wyłącz maszynę writualną, a następnie odłącz host od internetu.

# 2. Topologia sieci

Zbuduj sieć jak na obrazku poniżej:

![Alt text](/images/Screenshot%20from%202024-06-18%2021-36-00.png)

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
