# Window Subsystem Linux (WSL)

## Docker

- Install iptables, select 1

  ```bash
  sudo apt install -y iptables
  sudo update-alternatives --config iptables
  ```

- Add Docker's official GPG key

  ```bash
  sudo apt-get update
  sudo apt-get install ca-certificates curl
  sudo install -m 0755 -d /etc/apt/keyrings
  sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  sudo chmod a+r /etc/apt/keyrings/docker.asc
  ```

- Add repo to apt sources

  ```bash
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
  ```

- Install docker

  ```bash
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  sudo usermod -aG docker $USER
  sudo addgroup docker
  ```

- Delete the existing symbolic link or file for the resolution configuration and create a fresh configuration file

  ```bash
  sudo rm -f /etc/resolv.conf
  sudo nano /etc/resolv.conf
  ```

- Add these 2 lines:

  ```bash
  nameserver 8.8.8.8
  nameserver 1.1.1.1
  ```

- Stop resolv.conf from regenerating at boot. Edit /etc/wsl.conf:

  ```bash
  sudo nano /etc/wsl.conf
  ```

- Add the following 2 lines to it:

  ```bash
  [boot]
  systemd=true
  
  # Added these 2 lines:
  [network]
  generateResolvConf = false
  ```

- Exit and shutdown WSL

  ```bash
  exit
  ```

  ```powershell
  wsl --shutdown
  ```

- Start wsl

  ```powershell
  wsl -d Ubuntu
  ```

- Restart Docker

  ```bash
  sudo systemctl restart docker
  ```

- Try docker by running

  ```bash
  docker run hello-world
  ```

- References:
  - [daniel.es](https://daniel.es/blog/how-to-install-docker-in-wsl-without-docker-desktop/)
