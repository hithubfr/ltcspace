# Installing Litecoin Space on macOS

This guide provides step-by-step instructions for installing and running Litecoin Space (ltcspace) on macOS.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Methods](#installation-methods)
  - [Method 1: Docker Installation (Recommended)](#method-1-docker-installation-recommended)
  - [Method 2: Manual Installation](#method-2-manual-installation)
- [Troubleshooting](#troubleshooting)

## Prerequisites

Before installing Litecoin Space, ensure you have the following installed on your macOS system:

### 1. Install Homebrew

If you don't have Homebrew installed, install it first:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. Install Git

```bash
brew install git
```

### 3. Install Node.js and npm

Litecoin Space requires Node.js v16.16.0 and npm 7+. We recommend using `nvm` (Node Version Manager) to manage Node.js versions:

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Restart your terminal or run:
source ~/.zshrc  # or ~/.bash_profile if using bash

# Install the required Node.js version
nvm install 16.16.0
nvm use 16.16.0
nvm alias default 16.16.0
```

Verify the installation:

```bash
node --version  # Should show v16.16.0
npm --version   # Should show 7.x or higher
```

## Installation Methods

Choose one of the following installation methods based on your needs.

### Method 1: Docker Installation (Recommended)

This is the easiest method and is recommended for most users.

#### 1. Install Docker Desktop for Mac

Download and install Docker Desktop from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)

Or install via Homebrew:

```bash
brew install --cask docker
```

Start Docker Desktop from your Applications folder.

#### 2. Clone the Repository

```bash
git clone https://github.com/hithubfr/ltcspace.git
cd ltcspace
```

#### 3. Set Up Litecoin Core

You need a running Litecoin Core node. Install it using Homebrew:

```bash
brew install litecoin
```

Create a Litecoin configuration file at `~/Library/Application Support/Litecoin/litecoin.conf`:

```ini
txindex=1
server=1
rpcuser=mempool
rpcpassword=mempool
rpcallowip=127.0.0.1
rpcallowip=172.27.0.0/16
```

Start Litecoin Core:

```bash
litecoind -daemon
```

Wait for the blockchain to sync (this can take several hours or days).

#### 4. Configure Docker Compose

Navigate to the docker directory:

```bash
cd docker
```

Edit `docker-compose.yml` to configure your setup. At minimum, update the RPC credentials if you changed them in step 3.

#### 5. Start Litecoin Space

```bash
docker-compose up -d
```

Your Litecoin Space instance should now be running at [http://localhost](http://localhost)

To view logs:

```bash
docker-compose logs -f
```

To stop the services:

```bash
docker-compose down
```

### Method 2: Manual Installation

This method is recommended for developers or users who want more control over their setup.

#### 1. Clone the Repository

```bash
git clone https://github.com/hithubfr/ltcspace.git
cd ltcspace
```

#### 2. Install and Configure Litecoin Core

```bash
brew install litecoin
```

Create a configuration file at `~/Library/Application Support/Litecoin/litecoin.conf`:

```ini
txindex=1
server=1
rpcuser=mempool
rpcpassword=mempool
```

Start Litecoin Core:

```bash
litecoind -daemon
```

#### 3. Install and Configure MariaDB

Install MariaDB:

```bash
brew install mariadb
```

Start MariaDB:

```bash
brew services start mariadb
```

Create the database and user:

```bash
mysql -u root
```

In the MySQL prompt, run:

```sql
CREATE DATABASE mempool;
GRANT ALL PRIVILEGES ON mempool.* TO 'mempool'@'localhost' IDENTIFIED BY 'mempool';
FLUSH PRIVILEGES;
EXIT;
```

#### 4. Install Electrum Server (Optional but Recommended)

Address lookup functionality requires an Electrum Server. You can use Fulcrum or electrs-ltc:

**Option A: Fulcrum (Recommended)**

```bash
brew install fulcrum
```

Configure Fulcrum to connect to your Litecoin Core node. See [Fulcrum documentation](https://github.com/cculianu/Fulcrum) for detailed configuration.

**Option B: electrs-ltc**

Follow the [electrs-ltc installation guide](https://github.com/rust-litecoin/electrs-ltc).

#### 5. Set Up the Backend

Navigate to the backend directory:

```bash
cd backend
```

Install dependencies:

```bash
npm install
```

Create the configuration file:

```bash
cp mempool-config.sample.json mempool-config.json
```

Edit `mempool-config.json` and update the following settings:

- Set your Litecoin Core RPC credentials under `CORE_RPC`
- Set `BACKEND` to `"electrum"`, `"esplora"`, or `"none"` depending on your Electrum Server setup
- Configure `ELECTRUM` settings if using an Electrum Server
- Verify `DATABASE` settings match your MariaDB configuration

Build the backend:

```bash
npm run build
```

Start the backend:

```bash
npm run start
```

The backend should now be running on port 8999.

#### 6. Set Up the Frontend

Open a new terminal window and navigate to the frontend directory:

```bash
cd ltcspace/frontend
```

Install dependencies:

```bash
npm install
```

Configure the frontend:

```bash
npm run config:defaults:mempool
```

For development with local backend:

```bash
npm run serve
```

For production build:

```bash
npm run build
```

The built files will be in the `dist/` directory. You can serve these with any web server (nginx, Apache, etc.).

The frontend will be available at [http://localhost:4200](http://localhost:4200)

## Troubleshooting

### Common Issues

#### Node.js Version Issues

If you encounter errors related to Node.js version:

```bash
nvm install 16.16.0
nvm use 16.16.0
```

#### MariaDB Connection Issues

Ensure MariaDB is running:

```bash
brew services list | grep mariadb
```

If it's not running:

```bash
brew services start mariadb
```

#### Litecoin Core Not Syncing

Check the sync status:

```bash
litecoin-cli getblockchaininfo
```

The `verificationprogress` field should be close to 1.0 when fully synced.

#### Port Conflicts

If port 80 or 4200 is already in use, you can change the ports in the configuration files:

- For Docker: Edit `docker-compose.yml`
- For manual installation: Edit backend `mempool-config.json` and frontend `angular.json`

#### Permission Errors

If you encounter permission errors, ensure the cache and data directories are writable:

```bash
# In the backend directory
chmod -R 755 cache
```

#### Memory Issues

For large-scale deployments, you may need to increase Node.js memory:

```bash
# Start backend with more memory
node --max-old-space-size=4096 dist/index.js
```

### Getting Help

- Check the [main README](README.md) for general information
- Visit the [backend README](backend/README.md) for backend-specific documentation
- Visit the [frontend README](frontend/README.md) for frontend-specific documentation
- Review the [Docker README](docker/README.md) for Docker-specific information

### Updating Litecoin Space

To update to the latest version:

```bash
cd ltcspace
git pull
cd backend
npm install
npm run build
cd ../frontend
npm install
npm run build
```

For Docker installations:

```bash
cd ltcspace/docker
docker-compose pull
docker-compose up -d
```

## Next Steps

Once your Litecoin Space instance is running:

1. Access the web interface at http://localhost (Docker) or http://localhost:4200 (manual)
2. Wait for the mempool to populate with transactions
3. Explore blocks, transactions, and mining statistics
4. Configure additional features like mining pool tracking

Enjoy your self-hosted Litecoin Space instance!
