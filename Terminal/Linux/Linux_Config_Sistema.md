# ⚙️ Linux — Configuração do Sistema via Terminal

> Comandos para configurar o sistema no Zorin OS (base Ubuntu/Debian).

---

## 📦 Pacotes e Atualizações (APT)

```bash
sudo apt update                            # atualiza lista de pacotes
sudo apt upgrade                           # atualiza pacotes instalados
sudo apt full-upgrade                      # atualiza incluindo dependências
sudo apt install nome-pacote               # instala pacote
sudo apt remove nome-pacote                # remove (mantém configs)
sudo apt purge nome-pacote                 # remove + configs
sudo apt autoremove                        # remove dependências órfãs
sudo apt search termo                      # busca pacote por nome/desc
apt list --installed                       # lista instalados
dpkg -l | grep nome                        # verifica se pacote está instalado
```

---

## 🔊 Áudio (PulseAudio / PipeWire)

```bash
# Listar dispositivos de saída
pactl list sinks short

# Listar dispositivos de entrada
pactl list sources short

# Definir saída padrão
pactl set-default-sink <nome-do-sink>

# Definir entrada padrão
pactl set-default-source <nome-do-source>

# Volume da saída (0–150%)
pactl set-sink-volume @DEFAULT_SINK@ 80%
pactl set-sink-volume @DEFAULT_SINK@ +10%    # aumenta 10%
pactl set-sink-volume @DEFAULT_SINK@ -10%    # diminui 10%

# Mutar / desmutar saída
pactl set-sink-mute @DEFAULT_SINK@ toggle

# Reiniciar PulseAudio
pulseaudio --kill && pulseaudio --start
```

### Bluetooth — perfis de áudio

```bash
# Ver perfis disponíveis do fone
pactl list cards | grep -A 20 "bluez"

# Modo headset (entrada + saída — microfone ativo)
pactl set-card-profile bluez_card.<MAC> headset-head-unit

# Modo A2DP (somente saída — melhor qualidade de áudio)
pactl set-card-profile bluez_card.<MAC> a2dp-sink
```

> Substitua `<MAC>` pelo endereço do dispositivo com `_` no lugar de `:` (ex: `84_AC_60_D7_B7_A4`).

---

## 🖥️ Monitor e Resolução (xrandr)

```bash
xrandr                                     # lista monitores e resoluções disponíveis
xrandr --output HDMI-1 --mode 1920x1080    # define resolução
xrandr --output HDMI-1 --rate 144          # define taxa de atualização
xrandr --output HDMI-1 --off               # desliga monitor

# Dois monitores — lado a lado
xrandr --output eDP-1 --primary \
       --output HDMI-1 --right-of eDP-1
```

---

## 🌐 Rede

```bash
ip a                                       # lista interfaces e IPs
ip link show                               # estado das interfaces
nmcli device status                        # status de rede (NetworkManager)
nmcli connection show                      # lista conexões salvas
nmcli connection up "nome-wifi"            # conecta a uma rede
nmcli connection down "nome-wifi"          # desconecta

ping -c 4 google.com                       # testa conectividade
curl ifconfig.me                           # exibe IP público
ss -tulpn                                  # portas em uso
```

---

## 🔥 Firewall (UFW)

```bash
sudo ufw status                            # estado atual
sudo ufw enable                            # ativa firewall
sudo ufw disable                           # desativa

sudo ufw allow 22                          # libera SSH
sudo ufw allow 8080/tcp                    # libera porta TCP
sudo ufw deny 3306                         # bloqueia porta
sudo ufw delete allow 8080/tcp             # remove regra
sudo ufw reset                             # reseta todas as regras
```

---

## ⚡ Energia e Hardware

```bash
# Informações do sistema
uname -a                                   # kernel e arquitetura
lsb_release -a                             # versão do SO
hostnamectl                                # hostname e info do sistema
lscpu                                      # detalhes do processador
free -h                                    # memória RAM
df -h                                      # uso de disco
lsblk                                      # dispositivos de bloco
lsusb                                      # dispositivos USB
lspci                                      # dispositivos PCI

# Bateria / energia
upower -i /org/freedesktop/UPower/devices/battery_BAT0
cat /sys/class/power_supply/BAT0/capacity  # porcentagem da bateria

# Temperatura
sensors                                    # requer lm-sensors
sudo apt install lm-sensors && sudo sensors-detect
```

---

## 👤 Usuários e Grupos

```bash
whoami                                     # usuário atual
id                                         # UID, GID e grupos
groups                                     # grupos do usuário atual
sudo adduser novo-usuario                  # cria usuário
sudo usermod -aG sudo novo-usuario         # adiciona ao grupo sudo
sudo deluser nome-usuario                  # remove usuário
passwd                                     # muda senha do usuário atual
sudo passwd nome-usuario                   # muda senha de outro usuário
```

---

## 🔄 Serviços (systemd)

```bash
systemctl status nome-servico              # estado do serviço
systemctl start nome-servico               # inicia
systemctl stop nome-servico                # para
systemctl restart nome-servico             # reinicia
systemctl enable nome-servico              # habilita na inicialização
systemctl disable nome-servico             # desabilita na inicialização
systemctl list-units --type=service        # lista todos os serviços
journalctl -u nome-servico -f              # logs em tempo real
```

---

## 🐳 Docker (pós-instalação)

```bash
# Adicionar usuário ao grupo docker (evita sudo)
sudo usermod -aG docker $USER
newgrp docker                              # aplica sem reiniciar

# Testar
docker run hello-world
```

---

## ⚡ Atalhos Úteis

| Comando | Descrição |
|---------|-----------|
| `sudo !!` | repete último comando com sudo |
| `history \| grep apt` | busca no histórico |
| `alias ll='ls -lh'` | cria atalho de comando |
| `echo $SHELL` | shell atual |
| `htop` | monitor de processos interativo |
| `ncdu` | analisador de disco interativo |
