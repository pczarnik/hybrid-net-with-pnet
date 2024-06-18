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

# 2. Tworzenie laboratorium w PNETLab

