### Instalacja Node-Red na Raspberry Pi

1. Łączymy się z Raspberry Pi za pomocą terminala SSH lub otwieramy terminal
2. Uruchamiamy zdalny gotowy skrypt, który zainstaluje Node-Red

> ```
> bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
> ```

1. Korzystając z Raspberry Pi trzeba uruchomić Node-Red z parametrem zwiększającym częstotliwość zwalniania pamięci

> ```
> node-red-pi --max-old-space-size=256
> node-red-stat
> ```

1. Aby Node-Red uruchamiał się automatycznie po starcie systemu należy użyć następującej komendy

> ```
> sudo systemctl enable nodered.service
> ```

wyłączenie autostartu za pomocą polecenia

> ```
> sudo systemctl disable nodered.service
> ```

1. Edytor jest dostępny teraz przez przeglądarkę http://<adres_hosta>:1880

Szczegóły instalacji na RPi i innych platformach w dokumentacji Node-Red https://nodered.org/docs/getting-started/

**Scenariusz**: Wysłanie zdarzenia OnClick z Smart Panelu do Node-Red i zmiana stanu pierwszego wyjścia relay przez Node Red.