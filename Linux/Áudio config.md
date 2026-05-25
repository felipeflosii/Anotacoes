## Microfone Bluetooth

##### Habilitar entrada de áudio do fone Bluetooth
```bash
pactl set-card-profile bluez_card.84_AC_60_D7_B7_A4 headset-head-unit
```

##### Desabilitar entrada de áudio do fone Bluetooth
```bash
pactl set-card-profile bluez_card.84_AC_60_D7_B7_A4 a2dp-sink
```
