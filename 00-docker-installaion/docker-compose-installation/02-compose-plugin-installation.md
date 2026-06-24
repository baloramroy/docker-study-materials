# Install Docker Compose plugin for Linux

Docker compose **plug in** install **automatically** during the time of **docker installation**. If it doesn't then proceed with the **below procedure**.

- This installs the **plugin version**, so you will use:
  ```
  docker compose ...
  ```

  >**Requirement:** Docker Engine + Docker CLI must already be installed.


## Two Ways to Install docker compose plugin

### 1. Install from Docker Repository (Recommended)


**i. Set up Docker repository**

- Follow the repo setup guide for your distribution:\
  `Ubuntu | Debian | CentOS | Fedora | RHEL` (Choose your distro and add Docker’s official repo.)


**ii. Install Compose plugin**

- For **Ubuntu & Debian**:
  ```bash
  sudo apt-get update
  sudo apt-get install docker-compose-plugin
  ```

- For CentOS, Fedora, RHEL (RPM-based):
  ```bash
  sudo yum update
  sudo yum install docker-compose-plugin
  ```

**iii. Verify install or not**

- check docker compose version
  ```bash
  docker compose version
  ```
  
**Note:** **Auto-updates** + easiest maintenance.


### 2. Manual Installation (Not Recommended)
Use this only if you **cannot use the repository**. Manual install does **NOT** auto-update.

**i. Download the binary**

- First Setup docker **config environment** variable
  ```bash
  DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
  ```
  Breaking down the syntax:
    ```bash
    VARIABLE=${PARAM:-DEFAULT_VALUE}
    ```
  
    * `${ }` - Variable expansion syntax
    * `PARAM` - The parameter/variable we're checking
    * `:-` - The operator that means "use DEFAULT_VALUE if PARAM is unset or null"
    * `DEFAULT_VALUE` - What to use if PARAM isn't set

- Creating the plugins directory
  ```bash
  mkdir -p $DOCKER_CONFIG/cli-plugins
  ```

- Download and save the docker compose file
  ```bash
  curl -SL https://github.com/docker/compose/releases/download/v5.0.0/docker-compose-linux-x86_64 \
  -o $DOCKER_CONFIG/cli-plugins/docker-compose
  ```

  **Optional adjustments**
    * Install for **all users** → use `/usr/local/lib/docker/cli-plugins`
    * Install **another version** → change `v5.0.0`
    * Use **another architecture** → change `x86_64`

**ii. Make it executable**

- For single user:
  ```bash
  chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
  ```

- For all users:
  ```bash
  sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
  ```

- Verify compose version
  ```bash
  docker compose version
  ```
