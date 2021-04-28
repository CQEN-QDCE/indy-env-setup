# Configurer de l'environnement Hyperledger Indy
![Alt text](/indy-logo.png?raw=true "Hyperledger Indy")

Ce référentiel explique comment configurer localement un grand livre Hyperledger Indy et son environnement dans un **but de développement**. Ces instructions peuvent être légèrement mises à jour, il est donc recommandé de visiter également [Hyperledger Indy SDK repository](https://github.com/hyperledger/indy-sdk) pour plus de détails. Ces instructions sont strictement réservées aux machines locales pour la configuration d'un environnement de développement. Il est fortement déconseillé d'utiliser ces instructions pour le déploiement d'un environnement de production, car cela pourrait entraîner de graves problèmes de sécurité.

Hyperledger Indy est un écosystème logiciel complexe et il est composé de 3 éléments principaux :
* [indy-node](https://github.com/hyperledger/indy-node)
* [indy-plenum](https://github.com/hyperledger/indy-plenum)
* [indy-sdk](https://github.com/hyperledger/indy-sdk)

*[Hyperledger Aries](https://github.com/hyperledger/aries) et [Hyperledger Ursa](https://github.com/hyperledger/ursa) sont des bibliothèques importantes pour le développement avec Hyperledger Indy et seront utilisées dans une phase ultérieure. Elles sont contextuellement hors de portée pour ce tutoriel particulier.*

### Indy Node
Indy Node est la partie serveur d'un grand livre distribué conçu spécialement pour l'identité décentralisée. Il comprend toutes les fonctionnalités permettant d'exécuter des nœuds (validateurs et/ou observateurs) qui fournissent un écosystème d'[identité auto-souveraine] (https://sovrin.org/) au-dessus d'un grand livre distribué.

### Indy Plenum
Indy Plenum met en œuvre le [Plenum Byzantine Fault Tolerant Protocol] (http://pakupaku.me/plaublin/rbft/5000a297.pdf). Plenum est le cœur de la technologie de grand livre distribué de Hyperledger Indy. En tant que tel, il fournit des fonctionnalités assez similaires à celles trouvées dans Hyperledger Fabric. Cependant, il est spécialement conçu pour être utilisé dans un système d'identité, alors que Fabric est d'usage général.

### Indy SDK
Il s'agit du SDK officiel de [Hyperledger Indy] (https://www.hyperledger.org/projects/hyperledger-indy), qui fournit une fondation basée sur un registre distribué pour [l'identité auto-souveraine] (https://sovrin.org/). Indy fournit un écosystème logiciel pour une identité privée, sécurisée et puissante, et le SDK Indy permet de développer des clients. L'artefact principal du SDK est une bibliothèque appelable en C ; il existe également des wrappers pratiques pour divers langages de programmation et l'outil Indy CLI.

# Development Environment

## MacOS

1. Installer Rust et rustup (https://www.rust-lang.org/install.html).
2. Installer les bibliothèques et utilitaires natifs requis (libsodium est ajouté avec l'URL à homebrew puisque la version<1.0.15 est requise)
   ```
   brew install pkg-config
   brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/65effd2b617bade68a8a2c5b39e1c3089cc0e945/Formula/libsodium.rb   
   brew install automake 
   brew install autoconf
   brew install cmake
   brew install openssl
   brew install zeromq
   brew install zmq
   ```
3. Configurer les variables d'environnement :
   ```
   export PKG_CONFIG_ALLOW_CROSS=1
   export CARGO_INCREMENTAL=1
   export RUST_LOG=indy=trace
   export RUST_TEST_THREADS=1
   ```
4. Configurer la variable OPENSSL_DIR : chemin vers la bibliothèque openssl installée
   ```
   for version in `ls -t /usr/local/Cellar/openssl/`; do
        export OPENSSL_DIR=/usr/local/Cellar/openssl/$version
        break
   done
   ```
5. Cloner et compiler la bibliothèque:
   ```
   git clone https://github.com/hyperledger/indy-sdk.git
   cd ./indy-sdk/libindy
   cargo build
   ```
6. Pour compiler le CLI, libnullpay, ou d'autres éléments qui dépendent de libindy :
   ```
   export LIBRARY_PATH=/path/to/sdk/libindy/target/<config>
   cd ../cli
   cargo build
   ```
7. Définissez vos variables d'environnement `DYLD_LIBRARY_PATH` et `LD_LIBRARY_PATH` au chemin de `indy-sdk/libindy/target/debug`. Vous pouvez les mettre dans votre `.bash_profile` pour les conserver.

### Note sur le fonctionnement des nœuds locaux

Dans le dossier 'indy-sdk' cloné depuis (https://github.com/hyperledger/indy-sdk.git), vous verrez un dossier 'ci'. Démarrez le pool de nœuds locaux sur 127.0.0.1:9701-9708 avec Docker en exécutant:

```
docker build -f ci/indy-pool.dockerfile -t indy_pool .
docker run -itd -p 9701-9708:9701-9708 indy_pool
```
#### Automated Build Alternative
You can also try automated build by cloning the (https://github.com/hyperledger/indy-sdk.git) repo and run `mac.build.sh` in the `libindy` folder.

## Linux (Ubuntu 16.04)

1. Installer Rust et rustup (https://www.rust-lang.org/install.html).
1. Installer les bibliothèques natives et les utilitaires requis:

   ```
   apt-get update && \
   apt-get install -y \
      build-essential \
      pkg-config \
      cmake \
      libssl-dev \
      libsqlite3-dev \
      libzmq3-dev \
      libncursesw5-dev
   ```
   
1. `libindy` nécessite la version moderne `1.0.14` de `libsodium` mais Ubuntu 16.04 ne supporte pas l'installation depuis le dépôt `apt`.
 Pour cette raison, il faut compiler et installer `libsodium` à partir des sources :
 ```
cd /tmp && \
   curl https://download.libsodium.org/libsodium/releases/old/unsupported/libsodium-1.0.14.tar.gz | tar -xz && \
    cd /tmp/libsodium-1.0.14 && \
    ./configure --disable-shared && \
    make && \
    make install && \
    rm -rf /tmp/libsodium-1.0.14
```

4. Compiler `libindy`

   ```
   git clone https://github.com/hyperledger/indy-sdk.git
   cd ./indy-sdk/libindy
   cargo build
   cd ..
   ```
   
**Note:** Le paquet debian `libindy`, installé depuis le dépôt apt, est lié statiquement avec `libsodium`. 
Pour une construction manuelle, ceci peut être réalisé en passant `--features sodium_static` dans la commande `cargo build`.
   
 
### Note sur le fonctionnement des nœuds locaux

Dans le dossier 'indy-sdk' cloné depuis (https://github.com/hyperledger/indy-sdk.git), vous verrez un dossier 'ci'. Démarrez le pool de nœuds locaux sur 127.0.0.1:9701-9708 avec Docker en exécutant:

```
docker build -f ci/indy-pool.dockerfile -t indy_pool .
docker run -itd -p 9701-9708:9701-9708 indy_pool
```

## Windows

### Environnement de compilation

1. Configurer une machine virtuelle Windows. Des images gratuites sont disponibles à l'adresse suivante : [ici](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)
1. Lancer la machine virtuelle  
1. Télécharger Visual Studio Community Edition 2017 (ces instructions fonctionnent également avec Visual Studio Professional 2017)
1. Cochez les cases pour le _Développement de bureau avec C++_ et le _Développement Linux avec C++_.
1. Dans la partie résumé sur le côté droit, cochez également la case _support C++/CLI_.
1. Cliquez sur installer
1. Télécharger git-scm pour Windows [ici] (https://git-scm.com/download/win)
1. Installez git pour Windows en utilisant :
   1. _Utiliser Git à partir de Git Bash uniquement_ pour ne pas modifier les paramètres de chemin de l'invite de commande.
   1. _Checkout tel quel, commiter les terminaisons de ligne de style Unix_. Vous ne devriez pas commiter quoi que ce soit de toute façon, mais juste au cas où...
   1. _Utiliser MinTTY_
   1. Cochez toutes les cases pour :
      1. Enable file system caching
      1. Enable Git Credential Manager
      1. Enable symbolic links
1. Télécharger rust pour Windows [ici] (https://www.rust-lang.org/en-US/install.html)
   1. Choisissez l'option d'installation *1*.

### Obtenir/compiler les dépendances

- Ouvrez une invite de commande Git Bash
- Changez les répertoires pour Downloads :
```bash
cd Downloads
```

- Clonez le dépôt _indy-sdk_ de github.
```bash
git clone https://github.com/hyperledger/indy-sdk.git
```

- Télécharger les dépendances préconstruites [ici] (https://repo.sovrin.org/windows/libindy/deps/)
- Extrayez-les dans le dossier _C:\BIN\x64_.
> L'endroit où vous les mettez n'a pas d'importance, du moment que vous vous souvenez de l'endroit où vous les mettez.
> les variables d'environnement dans ce chemin

- Si vous ne construisez pas les dépendances à partir des sources, vous pouvez passer à *Build*.

### Dépôts binaires

- https://www.npcglib.org/~stathis/downloads/openssl-1.0.2k-vs2017.7z
- https://download.libsodium.org/libsodium/releases/old/libsodium-1.0.14-msvc.zip

### Dépôt source

- http://www.sqlite.org/2017/sqlite-amalgamation-3180000.zip
- https://github.com/zeromq/libzmq

### Compiler sqlite

Télécharger http://www.sqlite.org/2017/sqlite-amalgamation-3180000.zip

Créez un projet de bibliothèque statique vide dans Visual Studio et ajoutez le fichier `sqlite.c` et 2 en-têtes de l'archive extraite.
extraites. Ensuite, il suffit de le construire.

### Compiler libzmq

Follow to http://zeromq.org/intro.
- Téléchargez les sources de la dernière version stable pour Windows. 
- Ouvrir `zeromq-x.x.x/builds/msvc/vs2015/libzmq.sln` avec Visual Studio
- Si nécessaire, changez les plates-formes de la solution en x64 (si vous travaillez sur une architecture x64).
- Dans la barre de menu principale, choisissez build->build libzmq.
- Si la construction du projet a réussi, deux fichiers `libzmq.dll` et `libzmq.lib` devraient apparaître. 
  dans le chemin `zeromq-x.x.x/bin/x64/Debug/vXXX/dynamic`.
- Renommer `libzmq.lib` en `zmq.lib`.

### Construire

- Obtenir les dépendances binaires (libamcl*, openssl, libsodium, libzmq, sqlite3).
- Mettez tous les *.{lib,dll} dans un répertoire et les en-têtes dans le sous-répertoire include/.
- Ouvrez une invite de commande Windows
- Configurer l'environnement MSVS pour privatiser les constructions 64 bits par l'exécution de `vcvars64.bat` :
  
  ```
  "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\"vcvars64.bat
  ```
  
  Notez que selon la version de Visual Studio, l'emplacement de vcvars64.bat peut être différent. Par exemple, il peut être
  `"C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64\vcvars64.bat"`  
- Exécuter `"C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvars64.bat"`
- Indiquez le chemin d'accès à ce répertoire à l'aide de variables d'environnement :
  - `set INDY_PREBUILT_DEPS_DIR=C:\BIN\x64`
  - `set INDY_CRYPTO_PREBUILT_DEPS_DIR=C:\BIN\x64`
  - `set MILAGRO_DIR=C:\BIN\x64`
  - `set LIBZMQ_PREFIX=C:\BIN\x64`
  - `set SODIUM_LIB_DIR=C:\BIN\x64`
  - `set OPENSSL_DIR=C:\BIN\x64`
- Définissez PATH pour trouver les .dlls :
  - `set PATH=C:\BIN\x64\lib;%PATH%`
- changez le répertoire en `indy-sdk/libindy` et lancez `cargo build` (vous pouvez vouloir ajouter `--release --target x86_64-pc-windows-msvc`
  à cargo)

### contournement d'openssl-sys

Si votre build Windows échoue à se plaindre de gdi32.lib, vous devez éditer

```
  ~/.cargo/registry/src/github.com-*/openssl-sys-*/build.rs
```

et ajouter

```
  println!("cargo:rustc-link-lib=dylib=gdi32");
```

à la fin de la fonction `main()`.

Ensuite, essayez de recompiler l'ensemble du projet.

### Exécuter les tests d'intégration

* Démarrer le pool de noeuds locaux sur `127.0.0.1:9701-9708` avec Docker :
 
  ```     
  docker build -f ci/indy-pool.dockerfile -t indy_pool .
  docker run -itd -p 9701-9709:9701-9709 indy_pool
  ```          
 
  Veuillez noter que ce mappage de port entre le conteneur et l'hôte local requiert
  la dernière version de Docker pour Windows (conteneurs linux) et un système Windows prenant en charge Hyper-V.
  
  Si vous utilisez une distribution Docker basée sur Virtual Box, vous pouvez utiliser le futur transfert de port de Virtual Box pour faire correspondre les ports de conteneurs 9701-9709 aux ports locaux 9701-9709. 
  de Virtual Box pour faire correspondre les ports 9701-9709 des conteneurs aux ports 9701-9709 locaux.
 
* Exécuter les tests
  
  ```
  RUST_TEST_THREADS=1 cargo test
  ```
