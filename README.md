# Hybrid network with PNETLab (PL)

Poniższa instrukcja może posłużyć do konfiguracji PNETLaba na hypervisorze VMWare.
Inne hypervisory nie są wspierane.

Zainstaluj najnowszą, darmową wersję VMWare Workstation Pro - po przejęcie VMWare przez Broadcom konieczne jest niestety posiadanie konta na tej platformie: https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware+Workstation+Pro

Jeśli posiadasz Linuxa, po instalacji VMWare konieczne jest wykonanie komendy
```
sudo chmod 666 /dev/vmnet*
```

Jeśli będziesz chcieć uruchomić ansible ze swojego laptopa, musisz mieć system Linux (Windows ani WSL nie jest wspierany).
Jeśli nie masz możliwości użycia Linuxa - zainstaluj ansible wewnątrz VMki PNETLab (instrukacja poniżej).

Jeśli posiadasz system Linux, zainstaluj ansible wykorzystując swój package-manager.
Doinstaluj moduły `cisco` przez ansible-galaxy
```
ansible-galaxy collection install cisco.ios
```
Sklonuj repo https://github.com/pczarnik/hybrid-net-with-pnet.git


## 1. Konfiguracja PNETLab

Maszyna wirtualna PNETLab wymaga WVMware Workstation, Player lub ESXi, ale poniższe laboratiorium było testowane wyłącznie na Workstation. Obraz PNETLab można pobrać ze strony https://pnetlab.com/pages/download.
Poniższa instrukcja testowana była na wersji `PNETLab_4.2.10`.
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

4. Pojawi się okno konfiguracji - wpisz dwukrotnie hasło `pnet`, a potem nic nie zmieniaj, przeklikaj enterem.
  Maszyna automatycznie się zrestartuje.

    ![](/images/vm_firstboot_config.png)

5. Po automatycznym zrestartowaniu maszyny, zanotuj jej adres IP przydzielony przez DHCP.

    ![](/images/vm_ipaddress.png)

    Jeśli adres nie zostanie automatycznie przydzielony:

    ![](/images/pnet_noip.png)

    zaloguj się do niej za pomocą `root/pnet` i ustaw statyczny adres IP z tej samej sieci, na której jest hypervisor/laptop:
    ```
    ip address add dev pnet0 <IP/MASK 192.168.0.5/24>
    ```

6. Sprawdź, czy maszyna jest dostępna pingując ją z hypervisora/laptopa:
    ```
    ping <IP 192.168.0.5>
    ```

7. Teraz możesz zalogować się do PNETLaba przez ssh - wykorzystaj terminal lub PUTTY - login i hasło takie same jak wcześniej: `root/pnet`. Następne komendy wykonuj po zalogowaniu się do VMki.

8. (Krok opcjalny - jeśli będziesz uruchamiał ansible z wewnątrz VMki PNETLAba) Zainstaluj `pip3` i `ansible` za pomocą komendy:
    ```
    curl -O https://bootstrap.pypa.io/pip/3.6/get-pip.py
    python3 get-pip.py
    pip install ansible paramiko
    ```

9. (Krok opcjalny - jeśli będziesz uruchamiał ansible z wewnątrz VMki PNETLAba) Sklonuj repozytorium:
    ```
    git clone https://github.com/pczarnik/hybrid-net-with-pnet.git
    ```

9. Zainstaluj `ishare2`, za pomocą komendy:
    ```console
    wget -O /usr/sbin/ishare2 https://raw.githubusercontent.com/ishare2-org/ishare2-cli/main/ishare2 && chmod +x /usr/sbin/ishare2 && ishare2
    ```

    Przy automatycznej konfiguracji ishare2 skorzystaj z domyślnych ustawień zmieniając jedynie `Check SSL certrificate` na `n`.

   ![](/images/ishare_installation.png)

10. Pobierz obrazy routera i switcha:
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

11. Sprawdź, czy PNETLab działa:

    - Sprawdź adres IP PNETLaba:
      ```
      ip address show pnet0
      ```
      Otwórz ten adres IP w przeglądarce i zaloguj się wybierając logowanie `offline` i używając domyślne `admin/pnet`. Możesz zmienić `Default Console` na `HTML Console` (umożliwi to dostęp do konsoli wirtualnych urządzeń przez przeglądarkę)

    - Stwórz nowy lab:

        ![](/images/pnet_newlab.png)

    - Dodaj nowy node:

        ![](/images/pnet_newnode.png)

    - Sprawdź, czy na liście znajdują się Router i Switch:

        ![](/images/pnet_newrouter.png)

    - Wybierz router, zmień nazwę na `VR1` i zapisz:

        ![](/images/pnet_newrouter2.png)

    - Uruchom router

        ![](/images/pnet_routerstart.png)


    - Poczekaj aż router się włączy i odczytaj jego port telnet

        ![](/images/pnet_routerstarted.png)

    - Na laptopie skorzystaj z telneta (przez terminal lub PUTTY) i zaloguj się do routera - `<IP PNETLaba>:<port routera>`. Równie dobrze możesz użyć konsoli HTML klikając dwukrotnie na router w PNETLabie.

    - Jeśli połączenie nie zostało odrzucone i po chwili pojawił się dialog konfiguracyjny routera to znaczy, że PNETLab działa.

12. Odłącz laptop/hypervisor od publicznego internetu - nie jest to konieczne, ale sugerowane.

13. Brawo - skonfigurowałeś PNETLaba. Czas na stworzenie sieci hybrydowej.

## 2. Topologia sieci

1. Zbuduj sieć fizyczną jak na obrazku poniżej:

    ![](/images/physnet.png)

    Router skonfiguruj klasycznie przez kabel konsolowy z innego komputera:
    ```
    Router>enable
    Router#conf t
    Router(config)#hostname R1
    R1(config)#interface GigabitEthernet 0/0
    R1(config-if)#ip address 192.168.0.1 255.255.255.0
    R1(config-if)#no shutdown
    R1(config-if)#exit

    R1(config)#ip domain-name EXAMPLE.LOCAL
    R1(config)#crypto key generate rsa
    The name for the keys will be: R1.EXAMPLE.LOCAL
    Choose the size of the key modulus in the range of 360 to 4096 for your
      General Purpose Keys. Choosing a key modulus greater than 512 may take
      a few minutes.

    How many bits in the modulus [512]: 2048
    % Generating 2048 bit RSA keys, keys will be non-exportable...
    [OK] (elapsed time was 2 seconds)

    R1(config)#line vty 0 4
    R1(config-line)#transport input ssh
    R1(config-line)#login local
    R1(config-line)#exit

    R1(config)#username admin password pass
    R1(config)#username admin privilege 15
    R1(config)#end
    ```

    Powyższe komendy dodały adres IP do fizycznego routera i włączyły ssh z loginem: admin/pass
    Możesz teraz zalogować się do routera przez ssh - nie potrzebujesz kabla konsolowego.

    Ustaw statyczne adresy IP na laptopie/hypervisorze i VMce PNETLaba (przez VMWare).

2.  W PNETLabie stwórz poniższą konfigurację

    ![](/images/pnet_virtualconfig.png)

    By połączyć ze sobą node'y wirtualnymi kablami użyj:

    ![](/images/pnet_joinnodes.png)

    By dodać "chmurkę" umożliwiającą dostęp na zewnątrz PNETLaba użyj:

    ![](/images/pnet_addnetwork.png)

    ![](/images/pnet_addnetwork2.png)

    Odczytaj port telneta dla routera VR1 - użyjesz go do konfiguracji routera.

3. Zaloguj się do routera VR1 przez telnet `<IP PNETLaba>:<port telneta>` (lub użyj konsoli HTML) i skonfiguruj go dokładnie tak samo jak router fizyczny, zmieniając hostname na `VR1` i adres IP na `192.168.0.2`.

4. Zaloguj się do routera VR1 przez ssh (login admin/pass) - sprawdź czy działają podstawowe komendy, spinguj router R1:

    ```
    ping 192.168.0.1
    ```

5. Brawo brawo! Skonfigurowałeś sieć hybrydową, którą logicznie można reprezentować w następujący sposób:

    ![](/images/hybrid_net.png)


## 3. Tworzenie loopbacków i konfiguracja OSPF za pomocą ansible

1. Jeśli na routerach R1 i VR1 ustawiłeś inne adresy IP niż wskazane w instrukcji lub użyłeś więcej routerów fizycznych i wirtualnych zmień pliki:

    - `inventory.cfg` - dodaj lub zmień hosty i IPki pod `[all]` zgodnie ze wzorem:
        ```
        <hostname> ansible_host=<ip address>
        ```

    - `vars.yaml` - dodaj lub zmień konfigurację loopbacków i OSPFa dla każdego ze zmienionych/dodanych routerów:
        ```yaml
        ip_interfaces_config:
          R1:
            - name: Loopback1
              ipv4:
                - address: 10.0.0.1/32
        ...
        ospf_config:
        R1:
            processes:
              - process_id: 1
                network:
                  - address: 192.168.0.0
                    wildcard_bits: 0.0.0.255
                    area: 0
                  - address: 10.0.0.0
                    wildcard_bits: 0.0.0.255
                    area: 0

        ```

2. Uruchom ansible-owy playbook:

    ```console
    ansible-playbook -i inventory.cfg config_routers.yaml
    ```

    Playbook ten uruchomi listę zadań na routerach wymienionych w `inventory.cfg`.
    Najpierw wyczyści ewentualnie istniejące loopbacki i konfigurację OSPFa, a następnie skonfiguruje routery zgodnie z plikiem `vars.yaml`, dodając loopbacki i OSPFa.
    Na koniec spinguje wszystkie loopbacki ze sobą, sprawdzając, poprawność konfiguracji dynamicznego routingu.

3. Brawo brawo brawo, skonfigurowałeś automatycznie routing dynamiczny w sieci hybrydowej.
Spróbuj dodać dodatkowy router wirtualny (a może też i fizyczny?) - pamiętaj o zmodyfikowaniu `inventory.cfg` i `vars.yaml`.

    ![](/images/hybrid_net2.png)



## 4. Możliwe problemy

1. Wersja 6.1.2 ansible'owej kolecji `ansible.netcommon` zawiera błędy, które nie pozwalją na tworzenie interfejsów. Należy zainstalować starszą wersję, np. 6.1.1.
