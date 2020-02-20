# Prerequisites

## Google Cloud Platform

This tutorial leverages the [Google Cloud Platform](https://cloud.google.com/) to streamline provisioning of the compute infrastructure required to bootstrap a Kubernetes cluster from the ground up. [Sign up](https://cloud.google.com/free/) for $300 in free credits.

[Estimated cost](https://cloud.google.com/products/calculator/#id=55663256-c384-449c-9306-e39893e23afb) to run this tutorial: $0.23 per hour ($5.46 per day).

> The compute resources required for this tutorial exceed the Google Cloud Platform free tier.

## Google Cloud Platform SDK

### Install the Google Cloud SDK

Follow the Google Cloud SDK [documentation](https://cloud.google.com/sdk/) to install and configure the `gcloud` command line utility.

Para instalarlo en mi ubuntu tuve que hacer lo siguiente
```
$ echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
$ sudo apt-get update && sudo apt-get install google-cloud-sdk
```

Desde Windows decargo el paquete de google-cloud-sdk, lo descomprimo y ejecuto el install.
```
C:\fs\google-cloud-sdk>install.bat
Welcome to the Google Cloud SDK!
Active code page: 65001

To use the Google Cloud SDK, you must have Python installed and on your PATH.
As an alternative, you may also set the CLOUDSDK_PYTHON environment variable
to the location of your Python executable.
Active code page: 437
```
Outch! Me falta python, asi que descargo la ultima version del 2.7.x de py en https://www.python.org/downloads/release/python-2714/, en mi caso lo instalo en C:\Python37_64\. (Si baje el 2.7, con el 3.x al menos al momento de escribir esto, tira error https://www.cluemediator.com/google-cloud-sdk-installation-failed-on-windows).
En mi caso creo una variable de entorno  CLOUDSDK_PYTHON con el valor de "C:\Python27\python.exe".

En mi caso despues de instalar tuve que agregar la ubicacion de donde dejamos google-cloud-sdk en la variable de entorno path.

```
The installer is unable to automatically update your system PATH. Please add
  C:\opt\google-cloud-sdk\bin
to your system PATH to enable easy use of the Cloud SDK Command Line Tools.
```


Verify the Google Cloud SDK version is 262.0.0 or higher:

Verifico desde el ubuntu
```
jsfrnc@DESKTOP-O2Q0294:~$ gcloud version
Google Cloud SDK 281.0.0
alpha 2020.02.14
beta 2020.02.14
bq 2.0.53
core 2020.02.14
gsutil 4.47
kubectl 2020.02.14
```
Verifico desde Windows
```
C:\Windows\system32>gcloud version
Google Cloud SDK 281.0.0
bq 2.0.53
core 2020.02.14
gsutil 4.47
```
### Set a Default Compute Region and Zone

This tutorial assumes a default compute region and zone have been configured.

If you are using the `gcloud` command-line tool for the first time `init` is the easiest way to do this:

```
gcloud init
```

Then be sure to authorize gcloud to access the Cloud Platform with your Google user credentials:

```
gcloud auth login
```

Al no tener ningun proyecto, cree el jefrnc-k8s-bootcamp y lo configuro como mi proyecto activo de gcloud.
```
gcloud projects create jsfrnc-k8s-bootcamp
Create in progress for [https://cloudresourcemanager.googleapis.com/v1/projects/jsfrnc-k8s-bootcamp].
Waiting for [operations/cp.6267932993052209052] to finish...done.
Enabling service [cloudapis.googleapis.com] on project [jsfrnc-k8s-bootcamp]...
Operation "operations/acf.305bc62a-db74-493a-8db9-00f4403a292b" finished successfully.

gcloud config set project jsfrnc-k8s-bootcamp
Updated property [core/project].
```


Next set a default compute region and compute zone:

```
gcloud config set compute/region us-west1
```

Set a default compute zone:

```
gcloud config set compute/zone us-west1-c
```

> Use the `gcloud compute zones list` command to view additional regions and zones.

## Running Commands in Parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. Labs in this tutorial may require running the same commands across multiple compute instances, in those cases consider using tmux and splitting a window into multiple panes with synchronize-panes enabled to speed up the provisioning process.

> The use of tmux is optional and not required to complete this tutorial.

![tmux screenshot](images/tmux-screenshot.png)

> Enable synchronize-panes by pressing `ctrl+b` followed by `shift+:`. Next type `set synchronize-panes on` at the prompt. To disable synchronization: `set synchronize-panes off`.


En ubuntu lo instale de la siguiente manera

```
wget https://github.com/tmux/tmux/releases/download/3.0a/tmux-3.0a.tar.gz  
tar -zxf tmux-*.tar.gz
cd tmux-*/
./configure
make && sudo make install
```

A list of all windows is shown on the status line at the bottom of the screen.

Below are some most common commands for managing Tmux windows and panes:

Ctrl+b c Create a new window (with shell)
Ctrl+b w Choose window from a list
Ctrl+b 0 Switch to window 0 (by number )
Ctrl+b , Rename the current window
Ctrl+b % Split current pane horizontally into two panes
Ctrl+b " Split current pane vertically into two panes
Ctrl+b o Go to the next pane
Ctrl+b ; Toggle between the current and previous pane
Ctrl+b x Close the current pane


Next: [Installing the Client Tools](02-client-tools.md)
