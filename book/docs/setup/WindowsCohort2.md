# Plutus Playground Setup on Windows

## Notes

This guide will show you how to set up a Plutus Playground in Debian running on WSL2 in Windows. It also contains specific instructions which are usefull for Plutus Pioneers. However it can be used by anyone who wants to run the Plutus Playground on Windows.

This guide will not require you to use an editor outside of debian or even docker. However, as these are popular options I have included optional sections for installation of VSCode editor, WIndows Terminal and Docker.

You can follow this guide for any Linux distribution. This guide will use Debian, and it is the one I recommend if you don't have a specific preference. If you choose another distribution, you will need to substitute some of the Debian Bash commands with the equivalent in your chosen distribution.

This guide will assume you know how to change directories in Debian. Please pay close attention to which directory each command is called from. It is clearly denoted before each command. Make sure you cd into the proper directory. It is also noted if you should be in Bash or Nix. The guide will tell you when you should run nix-shell and also when you should be in nix-shell.

## Legend

~$ this is your home directory in Debian 
[nix-shell ~$] this is your home directory inside of nix-shell in debian
C:\> this is using command prompt

## Install WSL2
  
Follow the steps here to install WSL2 on Windows
  https://docs.microsoft.com/en-us/windows/wsl/install-win10

In step 6 you will be asked to install a distribution of linux on your new WSL2. Choose Debian or any other distribution. This guide will be specific to Debian.

When Windows installs your Debian, you will choose a user name and password. You will need these, so choose something fitting.

## Prepare your linux

	~$ sudo apt-get update
	~$ sudo apt-get upgrade
	~$ sudo apt-get install wget vim git curl npm

## Install Nix

	~$ sudo curl -L https://nixos.org/nix/install | sh
	~$ sudo mkdir /etc/nix
	~$ sudo touch /etc/nix/nix.conf
	~$ sudo chmod 777 /etc/nix/nix.conf
	~$ echo "sandbox = false
	use-sqlite-wal = false
	system-features = kvm
	substituters        = https://hydra.iohk.io https://iohk.cachix.org https://cache.nixos.org/
trusted-public-keys = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= iohk.cachix.org-1:DpRUyj7h7V830dp/i6Nti+NEO2/nhblbov/8MW7Rqoo= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=" >> /etc/nix/nix.conf
	~$ sudo chmod 644 /etc/nix/nix.conf
	~$ bash

## Get plutus and plutus playground repositories

	~$ mkdir code
	~/code$ git clone https://github.com/input-output-hk/plutus
	~/code$ git clone https://github.com/input-output-hk/plutus-pioneer-program

## Build Plutus  
  
If you are a plutus pioneer, then first checkout the correct commit, if not you can skip to the next step (nix build -f default.nix ...). You can read more about why this is necessary in the optinal section for pioneers.

	~/code/plutus$ cat ~/code/plutus-pioneer-program/code/weekxx/cabal.project | grep -m 1 tag:
	~/code$ git checkout ${commit-hash-tag-from-above-output}
  
	~/code/plutus$ nix build -f default.nix plutus.haskell.packages.plutus-core.components.library

## Build plutus playground

	~/code/plutus$ nix-build -A plutus-playground.client
  	~/code/plutus$ nix-build -A plutus-playground.server

## Build plutus and plutus playground dependencies

	~/code/plutus$ nix-build -A plutus-playground.generate-purescript
	~/code/plutus$ nix-build -A plutus-playground.start-backend
	~/code/plutus$ nix-build -A plutus-pab
  
## Start Plutus server

	~/code/plutus$ nix-shell
	[nix-shell ~/code/plutus/plutus-pab$] plutus-pab-generate-purs
	[nix-shell ~/code/plutus/plutus-playground-server$] plutus-playground-generate-purs
	[nix-shell ~/code/plutus/plutus-playground-client$] plutus-playground-server -i 120s

## Start the client

	~/code/plutus$ nix-shell
	[nix-shell ~/code/plutus/plutus-playground-client$] npm run start
	
At this point you have everything you need to run plutus playground and the npm will tell you towards the end of its output where you can access it. Default is https://localhost:8009/, so open up your broser and head on over.

In the playground you can use the in browser editor to write code, compile and simulate scenarios. There are also very usefull links in the top right if you need some help getting started. I recommend using these guides for any further information on the specifics of the actual playground.
  
## (Optional) Start the repl

With cabal repl you can run GHCi (a sort of cli into the haskell) directly in your terminal. It is very usefull for compiling your code if you are editing it in your Debian environment with vim or any other editor, such as a connected VSCode in Windows. It is also necessary to run the repl if you want to follow along in the plutus pioneer lessons.

In the examples below replace the final directory ( .. weekxx) with the correct week name if you are following the pioneer program or another directory with your code.

	~/code/plutus$ nix-shell
	[nix-shell ~/code/plutus-pioneer-program/code/weekxx$] cabal update
	[nix-shell ~/code/plutus-pioneer-program/code/weekxx$] cabal build
	[nix-shell ~/code/plutus-pioneer-program/code/weekxx$] cabal repl
  
## (Plutus Pioneers Only) Switching week

In the plutus pioneer program, each week you will have to adjust so you are using the correct commit of plutus. You need to match the correct Plutus repository commit hash for the weeks excercises.
  
Stop the client, server and repl if they are running. 

We will start by updating plutus and plutus pioneer program repositories, and then we will match the plutus commit to the one from the week we want to work on. Finally we will rebuild plutus.

	~/code/plutus$ git pull origin master
	~/code/plutus-pioneer-program$ git pull origin master
  
	~/code/plutus-pioneer-program/code/weekxx$ cat cabal.project | grep -m 1 tag:
	~/code/plutus$ git checkout ${commit-hash-tag}
	
	~/code/plutus$ nix build -f default.nix plutus.haskell.packages.plutus-core

Start the server, client and repl again


## (Optional) Install VSCode Editor
If you prefer to run VSCode editor over native linux editors then you can use this guide to install VSCode and then connect it to your linux distribution.

Install VSCode

	https://code.visualstudio.com/download
  
Install "Remote - Development" extension pack [ms-vscode-remote.vscode-remote-extensionpack]

Install "Docker" extension [ms-azuretools.vscode-docker]

Install "Haskell Syntax Highlighting" extension [justusadam.language-haskell]

	~/code$ code .

This will open VSCode with the directory tree in your ~/code directory. Anything inside will be accessible throught the editor. Saving, creating files, deleting files should all work seamlessly between VSCode and Debian at this point.


## (Optional) Install Windows Terminal
  
If you arent familiar with Windows Terminal, it is a program which lets you run a tabbed interface for your CLI programs, such as WSL2 instances, Powershell, Azure and Command Prompt. It will also let you configure colors, fonts and set other conveniant settings.
  
Install Windows Terminal

	https://docs.microsoft.com/en-us/windows/terminal/get-started
  
Inside of Windows Terminal, open up settings and add a new profile pointing to your debian executable (located in %programfiles%/WindowsApps/)
  
If your windows user doesnt have access to the %programfiles%/WindowsApps/ folder, then you neeg to give yourself permission. In a command prompt with administrative privilages

	C:\WINDOWS\system32> takeown /f "%programfiles%\WindowsApps"
	C:\WINDOWS\system32> icacls "%programfiles%/WindowsApps" /grant %username%:RX


## (Optional) Install Docker Desktop for Windows

You do not need to use Docker if you follow this guide. However, as it is a popular option, I will mention how to do it here. 

Follow the instructions located below to install Docker for Windows

  https://docs.docker.com/docker-for-windows/install/
  
Launch Docker and make sure it has WSL2 enabled for your distribution (you will find this in Docker settings).
  
Before docker load, verify that docker works with.

	~$ docker run hello-world

If you get an error, then try installing docker-load and also adding your user to the docker group
	
	~$ sudo apt-get install docker-load

	~$ sudo groupadd docker
	~$ sudo usermod -aG docker ${user-name}
  
Load the plutus Docker

	~/plutus$ docker load < $(nix-build default.nix -A devcontainer)
	
The docker will take some time to load. But if all goes well and you don't receive any more error messages, you are ready to go.
