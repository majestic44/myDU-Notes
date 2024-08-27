## 🚀 What you will need
----------------------------------------------------------------
A Linux server with docker and docker-compose installed, or a windows PC with docker-desktop installed. If you plan on hosting an instance on the public internet, having a domain name and going through the “securing your stack” steps a few sections below is highly recommended. In case you are using Windows, replace all '.sh' scripts invocation with '.bat' equivalent script in this whole documentation.

## 🌞 How big should my server be?
----------------------------------------------------------------
Default install requires 20Gb of disk space, and takes 3Gb of database size (mongodb + postgresql). This will grow with the number of constructs, blueprints and holes in the planets that are made. CPU-wise 4 to 8 cores should (no promises) work for a small bunch of players. Please note that industry units are quite CPU hungry, so you might need to beef up if you plan on making huge factories.

## 🏦 Architecture overview
----------------------------------------------------------------
> [!IMPORTANT]
> All exposed services except the front which is a direct GRPC connection are routed through a nginx HTTP server, to expose only public routes. A player client first connects to "queueing", the queuing service, which validates login and redirects player to a "front" with rate limiting.

#### The client then connects over GRPC to the front. Most traffic is routed on that connection, except:
- voxel and mesh gets are sent directly to the voxel service
- construct elements are fetched directly from orleans
- some element properties are fetched directly over http (served by nginx)
#### The stack runs the following images from a docker-compose file:
- `nginx` with mounted conf: ensure only public routes are accessible
- `mongo` with mounted data dir: voxels and meshes are stored there
- `redis` used internally for caching and inter-service data exchange
- `postgres with mounted data dir` main databases for everything not voxel
- `rabbitmq` use for inter-service internal communication
- `zookeeper` needed by kafka
- `kafka` used for internal job queueing
- `front` endpoint for client, mainly just a router to other services
- `node` visibility service
- `orleans` runs the Microsoft Actor Framework with same name. Everything gameplay is in there
- `constructs` microservice that serves construct information
- `queueing` authenticates and queues clients before dispatching them to the front
- `voxel` everything voxel and mesh
- `market` market microservice handling market orders and transactions
- `nodemanager` used for proper stack initialisation
- `backoffice` the backoffice used by game masters and support team
- `sandbox` image containing various maintenance utilities

> [!NOTE]
> All DU services use a YAML configuration file, mounted as `/config/dual.yaml`. Additionally some resources used at runtime are stored in `data/` subdirectories. Those can be moved to an AWS S3 bucket by changing the configuration.

#### Runtime resource folders:
- `user_content(RW)` those are element properties that can be too big to store efficiently in postgres.
- `tutorials` startup choices, challenges and tutorials
- `pools` ore pool definitions for planets and alien space cores
- `wrecks` wreck models
- `asteroids` asteroid models\
- `aliens` alien space core definitions

> [!NOTE]
> Finally, database files (mongo and postgres) are stored in `db-mongo` and `db-postgres` locally. You can change the data and db path by editing the `.env` file.

## 🪟 Quickstart for Windows
----------------------------------------------------------------
Install Docker Desktop from Docker Inc. It is free and does not require any signup. Start `dual-universe-server-installer.exe` . Do **not** pick a location in `Program Files` for the installation, the default is fine if you have enough disk space. The installer takes 20G of disk space and might need an hour to complete. Once it’s finished running, double-click on `Start DU` icon on your desktop to launch the server, and connect to it with the MyDU client, putting `localhost` in the `server URL` field, and the credentials you gave to the installer, using “admin” as login.

#### ⬇️ Making your server accessible to others
----------------------------------------------------------------
If you wish to host a DU server for others on your home internet connection in insecure mode, you need to forward a few TCP ports from your Router to your computer. You can either use the UI of your Router, or a program like `UPnP Wizard` . You need to forward TCP ports ` 8081 ` , ` 10111 ` , ` 9630 ` , ` 9210 ` and ` 10000 ` . Finally, you need to edit config/dual.yaml so that it advertises your public IP address: first obtain your public ip address (ask any search engine), then run the below command, replacing the two occurrences of `MY_IP` with your actual IP address. And that's it, other people should be able to connect to your DU server by entering your IP address in the `server URL` field.

```python
python3 scripts\\config-set-domain.py config/dual.yaml htt p://MY_IP MY_IP
```

#### 🦌 Quickstart without installer
----------------------------------------------------------------
Download the compose and scripts package provided by Novaquark. This is achieved by running in the directory where you want to copy files:

```docker
docker run --rm -it -v ./:/output novaquark/dual-server-fas tinstall:latest
```

All scripts must be run from this directory. On windows replace all '.sh' scripts with their '.bat' version with same name. Run the script that will edit the config/dual.yaml file by filling your domain name or IP address to the components that need absolute URLS:

```docker
cp config/dual.yaml config/dual.yaml.ori python3 scripts/config-set-domain.py config/dual.yaml htt p:// MY_IP
```

Second argument must be a URL, third argument must be an IP address.

Set password for your first admin user:

```docker
scripts/admin-set-password.sh admin MY_PASSWORD
```

Start the complete stack:

```docker
scripts/up.sh
```

And that's it, you should be able to log in as admin to the game and backoffice. Do NOT run `docker-compose up -d` as inter-service dependency will make some services fail to boot properly.

#### 📀 Backing up your stack
----------------------------------------------------------------
A backup script is provided as `scripts/backup.sh` ( `backup.bat` for Windows). When run it will create a directory with current date, and dump postgres, mongo and user_content in this directory, while the server stack is running.

## 🛠 Maintenance mode
----------------------------------------------------------------
If you set in `dual.yaml` `queueing.start_maintenance` to true, the server will boot in maintenance mode, allowing only admins to connect. You can exit maintenance mode with:

```
scripts/maintenance-mode-off.sh
```

## 💥 Known Issues and Quirks
----------------------------------------------------------------
Admin players (flagged as such in the backoffice) are intentionally invisible to non admin players. They can chose to be visible by toggling a setting in the “global settings” client menu.

## 👱 User management
----------------------------------------------------------------
The oauth and community database management used by Novaquark has been replaced by a simple local auth system (with securely hashed passwords). Users are stored in the `dual.auth` table. Notable fields are username (login), password (hashed) and roles (comma-separated list of roles). The `config/dual.yaml` can be edited to tweak which role can do what, in the `auth.roles` section. `game_access` specifies the roles to login in game: a user must have one of these roles to login with the client and play DU. `high_priority` specifies roles which connects in priority from a separate wait queue `bypass_priority` roles bypass the waiting queue entirely. `impersonation` are allowed to impersonate other users by using the `login: impersonate_to` syntax. The role `bo` is required, but not sufficient, to connect to the Backoffice: the backoffice also limits which action a user can take based on it's own role system.

Those roles can be tweaked with dual.yaml entries
backoffice.administrator_roles,
`backoffice.customer_support_roles` and `backoffice.game_designer_roles` . Users can be managed on the backoffice `Users` page, which require the role `Administrator` to access. 

Users can reset their own password by going on the `/guest` page of the backoffice. For mails to work properly you need to fill a few fields on the backoffice section of the dual.yaml config file:

- `public_url` this is the base public url at which your backoffice is reachable
- `smtp_sender_email` this is the 'from' email used to send the password reset email
- `smtp_host` host of your SMTP server. The default 'smtp' uses the built-in postfix.

Once this is setup you can also invite new users by email or invite link from a form on the `Users` page.

#### 🧾 Automatic account creation
----------------------------------------------------------------
If you set `auth.allow_account_creation` to true in `dual.yaml` , players will be allowed to create accounts on their own on your server without any supervision by going to the `/guest` backoffice page.

## 📰 Public Server Listing
----------------------------------------------------------------
The myDU client has a `server list` feature that lists public myDU servers. You can opt-in to this system and have your server publicly listed from the backoffice by going to the `public listing` page on the left menu. You will be presented with a form to fill in to register your server and have it publicly listed. Once done you can return to this page to edit information associated with your server, such as name and description.The server list system remembers your server using the file `config/server_secret` . To unlist permanently one can simply delete this file. Please note that Novaquark requests that you adhere to the terms of service, and Novaquark reserves the right to remove your server listing for any reason.

## 🔐 Securing your stack: Enabling SSL
----------------------------------------------------------------
> [!WARNING]
> This is an advanced use case that requires a bit of knowledge in DNS handling. By default my-du uses multiple http connections which are unencrypted, which is not ideal. If you own a domain name, you can use the provided script `du-ssl.py` through it's wrapper `./scripts ssl.sh` to:

- obtain SSL certificates through `letsencrypt`
- reconfigure `dual.yaml` to advertise the new https domain-based urls
- reconfigure the `nginx` proxy to serve all ports with SSL

#### 🌍 Creating DNS entries
----------------------------------------------------------------
The first step is to create 5 `A Records` in your DNS zone pointing to the IP of your server. This is usually accomplished through connecting to your domain registrar website. By default they are named `du-orleans` , `du-queueing` , `du-usercontent` , ` du-voxel` and `du-backoffice` , but those can be renamed to something else (see next section).

#### ✍️ Editing domains.json
----------------------------------------------------------------
Open the file 'config/domains.json in a text editor, and fill it according to the domain names you've created. The simplest way is to set `prefix` to `du-` and `tld` to your domain name.

Alternatively you can set the full domain per service by creating a `services` dictionary entry and filling the values for `orleans` , `queueing` , `backoffice` , `voxel` and `usercontent` .

#### ↗️Redirecting ports
----------------------------------------------------------------
The ports `443` , and `80` (during install only) on those domain names must point to the computer running the my-du server. If you are behind a router such as a home connection, you probably need to configure your router to redirect those ports.

#### 🔑 Creating SSL certificates
----------------------------------------------------------------
Temporarily disable any service (apache, nginx) listening on port `80` of the computer running the my-du server, and type the following:

```
./scripts/ssl.sh --create-certs
```

This will start certbot from letsencrypt to request the certificate for the 5 domain names defined in `config/domains.json` . It might ask you a few questions on the terminal. Note that it will emit a challenge by connecting to port `80` on all five domains, so you might have to wait for DNS propagation before this step. If all went well the keys and certificates are created in the `letsencrypt` folder.

#### 🥞 Reconfiguring the stack
----------------------------------------------------------------
Simply run the below command, replacing `MY_IP` with the public IP address of the server or computer running the my-du server.

```
./scripts/ssl.sh --config-dual MY_IP
```

Then run this:

```
./scripts/ssl.sh --config-nginx
```

These two commands will update `config/dual.yaml` and `nginx/conf.d` with the domain information.

#### 🖋 Editing docker-compose to forward port 443
----------------------------------------------------------------
Open docker-compose.yml in a text editor, locate the `nginx` section and uncomment the line `- 443:443` to forward port `443` to the nginx server. You can comment all other port lines as they are unused in SSL mode.

#### 🌮 Wrapping up
----------------------------------------------------------------
Once SSL is enabled your stack only uses two ports: `443` and `9310`. Your stack server address becomes `https://du-queueing.MYDOMAIN:443`. The certificates are valid for 3 months and can be renewed by running:

```
./scripts/ssl.sh --update-certs
```

## 🔗Advanced: authentication webhook
----------------------------------------------------------------
Should you wish to interfere with the queueing service authentication mechanism, a webhook can be enabled:

Set in dual.yaml “auth: auth_hook_url: ” that url will be called with a json payload containing the login request without the password. Login will be denied if an HTTP error code (4xx 5xx) is returned. Note that the webhook is called by the queuing service itself, so from within the docker stack.

## 🕹 Game master guide
----------------------------------------------------------------
All element properties and many gameplay features are tweakable through the `item hierachy`, accessible from the backoffice. The backoffice is accessible on port `12000` on your host over https. It uses by default a self-signed certificate that you can change, stored in `nginx` directory.

#### 💿 Image management
----------------------------------------------------------------
By default the server will prevent your users from using images at external URLs. You as administrator can specify on which domains images are allowed to load through the `CoherentConfig` item bank entry, on field `imageCDNList`, that you can change on the backoffice. To allow all (at your own risks), use the wildcard `*`.

#### ⚛️ Basic element properties
----------------------------------------------------------------
Let's take an example: suppose you want to increase max HP of the container hub. To achieve this, login to the backoffice and go to the component hierarchy page through the left menu. Type `ContainerHub` in the search field, which should highlight a single entry in the hierarchy below. Clicking on the `ContainerHub` card should open a new page where you can edit properties.

The editor shows you the definition of a ContainerHub in YAML format. Change the value of `hitpoints` to your liking, and hit `Save` . Et voila, all newly deployed container hubs will have your new value of hit points set!

Existing elements are not affected: since the `hitpoints` property can be affected by talents, the actual max HP value is stored per element at deployment time in the database in what is called a dynamic property. Fortunately you can ask the system to recompute existing properties through an administrative command in the sandbox image:

```
/python/DualSQL/DualSQL --recompute-props DEF.json
```

`DEF.json` is a json dictionary of element names to list of property names that changed. You can use parent element names and it will recurse. So following our example, you'd create a DEF.json with the following content:

```
{ 
"ContainerHub": ["hitpoints"] 
}
```

And execute the property recomputation by running:

```
docker-compose run --rm --entrypoint /python/DualSQL/DualSQ L --mount \\ type=bind,src=$PWD/DEF.json,dst=/DEF.json sandbox --reco mpute-props /DEF.json
```

This will update the hitpoints property of all deployed container hubs, by taking the new base value and re-applying all talents that were used.

> [!NOTE]
>  It is highly recommended to backup your item hierarchy data when tinkering with it, by using the export button, which will download a YAML dump of the item hierarchy to your computer.

#### 🚗 Exporting blueprints from the production server
----------------------------------------------------------------
Should you wish to import a blueprint you own from the Dual Universe production server, you can use the meshexporter service here. Login with your NQ credentials. You will be presented with a list of all core blueprints in your inventory. Simply hit the “export” button to download a JSON representation of your blueprint. Then login to the backoffice of your my-du server, and go to the blueprint page. From there you can import the JSON file to generate the blueprint, and grant it to any player

#### 🪨 Dynamic asteroids
----------------------------------------------------------------
Those are controlled by `AsteroidManagerConfig` in `FeaturesConfig`:

- `count` : how many asteroids per tier to maintain
- `lifetimeDays` : how long before destroying an asteroid
- `notBefore` : unix timestamp that prevents the whole feature from activating before given date
- `planets` : list of planet ids around which to spawn asteroids

Asteroids are spawned from a list of asteroid model fixtures (the construct JSON files) located in `data/asteroids` on the host. An unlinked backoffice page `v3/asteroidsmanager.html` can be accessed once logged in to show information about live asteroids.

#### 🧠 Talent system
----------------------------------------------------------------
All talents are defined in the item hierarchy as direct children of the `talent` node. Let's consider this example and break down the fields:

```
ContainerProficiency:
	parent: Talent
	displayName: Container proficiency
	group: InventoryManager
	description: +10 % storage volume on containers when depl oyed
	order: 3
	requiredTalents:
	- {name: "NanopackUpgrades", level: 3}
	  costs: [1800, 9000, 45000, 225000, 1125000]
	  effects:
	  - name: ContainerProficiency
	    target: PLAYER_ITEM
	    modifiers:
	    - name: ContainerProficiency
	      property: maxVolume
	      action: ADD_PERCENT
	      value: 10
	      itemId: ItemContainerBase
```

`Group` specifies in which group the talent will appear. Talent groups are defined in the item hierarchy too.

`description` is what is shown to the player on the client

`requiredTalents` sets up the prerequisites to acquire level 1 of this talent

`costs` are the XP cost per level

`effects` is a list of effects, each effect being a list of modifiers.

`target` specifies the effect type. Possible values are:

- `PLAYER_ITEM` element properties when put down
- `PLAYER` affects a property of the player (see Character in item hierarchy)
- `ELEMENT` affects a property of an element while using it

`property` is the affected property. `itemId` is the affected item name (this item and all of it's children). `action` can be `ADD` or `ADD_PERCENT` to be additive or multiplicative. `value` is the bonus value per level.

#### 👽 Alien space constructs
----------------------------------------------------------------
Alien space cores are reconciled dynamically if needed at stack boot, from the definition in file `data/aliens/alienspacecores.json`. The construct model used is `data/aliens/alien-station.json`.

The definition file looks like:
```
{
	"990001": {"name": "Alpha", "position": {"x": 33946000, "y": 71381990, "z": 28850000}}, 
}
```

The key is the construct id. You must use the `9900XX` range. The ore pools provided by the alien stations are in `data/pools/orepools-0.json`.

#### 🏝 Territory fees
----------------------------------------------------------------
Territory fees are controlled by the `TerritoriesConfig` I.H. node. All fields should be self-explanatory.

#### 🛥 Inactive asset requisition
----------------------------------------------------------------
Being tied to the Novaquark Community database, it is recommended to disable the IAR for now as it will not work properly. This can be done by setting `constructGarbageCollection` to `false` in `FeaturesList`.

#### 👥 Organization construct limit
----------------------------------------------------------------
Can be disabled by setting `FeaturesList.orgConstructLimit` set to false. Can be set to warning only without enforcing any deletion by setting `orgConstructLimitWarnOnly` to true.

#### ⛏️ Ore pools
----------------------------------------------------------------
Ore pools are defined in json files in `data/pools` named `orepools-PLANETID.json` (0 being for alien space cores). Those files are cached at runtime, so changes require a stack reboot.

#### ⚔️ PvE
----------------------------------------------------------------
The pve mission definition files are located in `data/tutorials/sentinel`. `tiers.json` gives generic info about the mission tiers. `library.yaml` defines the ships, weapons, patrol areas The parameters of the dynamic mission generator are in the item bank under `PvEGenerationConfigTIER`. Finally, the construct models of NPC ships are in `ships` subdirectory.

#### 💰 Missions and jobs
----------------------------------------------------------------
Two backoffice page can be used to list jobs and missions: `v3/missions.html` and `v3/formalmissions.html`. Aphelia missions are defined in a YAML file, which can be injected in the unlinked backoffice page `v3/haulingfixtures.html`. A sample file is provided in the `data/extra` directory. The ratio of visible aphelia missions is configurable in the item bank through the `MissionConfig` object.

#### 📈 Market bot orders
----------------------------------------------------------------
Those are defined in `data/market_orders`. For each market the system will recurse the construct hierarchy until it finds a csv with name the id of the construct. To trigger bot order generation, go to the backoffice markets page, and hit the `seed` button at the top.

#### 🏢 Backoffice extra pages
----------------------------------------------------------------
`v3/wallets.html` can be used to see wallet logs and notification logs. Finally,
`v3/audit.html` can be used to see the backoffice audit log, which shows all operations made through the backoffice that modified the state of the game.

### Credits
NQ-Bearclaw
