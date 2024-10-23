# Creating a local testing and development setup via Docker

## Additional information for Windows users

If you want to use the local setup on a Windows machine, it seems advisable to use the Windows subsystem for Linux (
WSL).
An installation guide can be found [here](https://learn.microsoft.com/en-us/windows/wsl/install).

Also see this [additional information about using Docker in combination with WSL2](https://docs.docker.com/desktop/wsl/)

## Initial Setup

Make sure to have `openssl` and `jq` installed in your shell. openssl is pre-installed on most operating systems. jq can be installed via the usual installation repositories, see [here](https://jqlang.github.io/jq/download/)

Run the following script to generate the necessary keys. It will also create an .env file in the ./local folder.


```shell
cd local
sh generate-keys.sh
```


## Start

#### Infrastructure Setup

First start the infrastructure by running

```shell
docker compose -f docker-compose-infrastructure.yaml up
```


This will run several services on these respective ports:

1. **MIW (Managed Identity Wallet)**  
   - **Ports**: `8000` (mapped to `80`), `8090`  
   - Handles identity management.

2. **PostgreSQL**  
   - **Port**: `5432`  
   - Database for MIW.

3. **Keycloak**  
   - **Port**: `8080`  
   - Manages identity and access with SSO.

4. **Vault**  
   - **Port**: `8200`  
   - Manages secrets securely.

5. **Mock Util Service**  
   - **Port**: `8888`  
  

6. **BDRS**  
   - **Ports**: `8580`, `8581`, `8582`  
   

**Network**: All services are connected to `miw-net`.



After the MIW container has finished booting, use this script to initialise two wallets for customer and supplier:

```shell
sh init-wallets.sh
```


After starting the central infrastructure, initialize the bdrs-service. To do so, just run the script `seed-bdrs.sh` created during the run of script `generate-keys.sh`.

```shell
sh seed-bdrs.sh
```


### Setup EDC and DTR
Then start the Two EDC and Digital Twin Registry containers via:

```shell
docker compose up
```


This `docker-compose.yml` defines multiple services with their respective ports and dependencies:

1. **PostgreSQL (Customer DB)**
   - **Port**: `5433` (mapped to internal `5432`)

2. **EDC Customer Control Plane**
   - **Ports**: `8180`, `8181`, `8182`, `8183`, `8184`
   - Customer's control plane for managing data exchanges.

3. **EDC Customer Data Plane**
   - **Ports**: `8280`, `8283`, `8285`, `8299`
   - Data plane for customer-side data processing.

4. **Digital Twin Registry (DTR) Supplier**
   - **Port**: `4244` (mapped to internal `4243`)
   - Manages supplier digital twin data.

5. **EDC Supplier Control Plane**
   - **Ports**: `9180`, `9181`, `9182`, `9183`, `9184`, `1044`
   - Supplier's control plane for data management.

6. **EDC Supplier Data Plane**
   - **Ports**: `9280`, `9283`, `9285`, `9299`
   - Data plane for supplier-side data processing.

**Network**: All services are connected via `miw-net`, an external Docker network.

## Restarting Services

To stop all the containers, which are not part of the infrastructure, by deleting the volumes, i.e. run

```
docker compose down -v
```

restart via

```shell
docker compose up
```

In general, it is not necessary to restart the infrastructure,
However, in rare cases there may be issues with the MIW. If this happens, you should use the cleanup script as mentioned in the debugging section below and then repeat the above-mentioned steps beginning with the Initial Setup section.

## Notes on debugging

### Vault & Certs

When having problems with the certs or the vault, one may need to delete the vault container.
The following script stops all infrastructure containers.

```shell
cd local
sh cleanup.sh
```

Then start your containers again with the aforementioned commands. 

## NOTICE

This work is licensed under the [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0).

- SPDX-License-Identifier: Apache-2.0
- SPDX-FileCopyrightText: 2024 Contributors to the Eclipse Foundation
- Source URL: https://github.com/eclipse-tractusx/puris
