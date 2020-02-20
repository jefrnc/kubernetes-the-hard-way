# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial: [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).


## Install CFSSL

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

__Tip__:
  > PKI son los siglas de Public Key Infrastructure (infraestructura de clave pública), o lo que es lo mismo, todo lo necesario, tanto de hardware como de software, para las comunicaciones seguras mediante el uso de certificadas digitales y firmas digitales. De esta manera se alcanzan los cuatro objetivos de la seguridad informática : autenticidad, confidencialidad, integridad y no repudio.

- cfssl : Es una herramienta de línea de comandos y un servidor API HTTP para firmar, verificar y agrupar certificados TLS.
- cfssljson :utiliza la salida JSON de los programas cfssl y multirootca y escribe certificados, claves, CSR y bundles  a disco.

Download and install `cfssl` and `cfssljson`:

### OS X

```
curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/darwin/cfssl
curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/darwin/cfssljson
```

```
chmod +x cfssl cfssljson
```

```
sudo mv cfssl cfssljson /usr/local/bin/
```

Some OS X users may experience problems using the pre-built binaries in which case [Homebrew](https://brew.sh) might be a better option:

```
brew install cfssl
```
 

### Linux

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssljson
```

```
chmod +x cfssl cfssljson
```

```
sudo mv cfssl cfssljson /usr/local/bin/
```


### Windows


En Windows, descargue el ultimo release https://github.com/cloudflare/cfssl/releases y lo aloje en C:\opt\cfssl, cargue la ruta en la variable de entorno PATH.

```
C:\opt\cfssl>dir /b
cfssl-bundle.exe
cfssl-certinfo.exe
cfssl-newkey.exe
cfssl-scan.exe
cfssl.exe
cfssljson.exe
mkbundle.exe
multirootca.exe
```

### Verification

Verify `cfssl` and `cfssljson` version 1.3.4 or higher is installed:

En Mac o Linux
```
cfssl version
```
```
Version: 1.3.4
Revision: dev
Runtime: go1.13
```

```
cfssljson --version
```
```
Version: 1.3.4
Revision: dev
Runtime: go1.13
```

En Windows
```
C:\opt\cfssl>cfssl version
Version: 1.4.1
Runtime: go1.12.12

C:\opt\cfssl>cfssljson --version
Version: 1.4.1
Runtime: go1.12.12
```
## Install kubectl

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

### OS X

```
curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/darwin/amd64/kubectl
```

```
chmod +x kubectl
```

```
sudo mv kubectl /usr/local/bin/
```

### Linux

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl
```

```
chmod +x kubectl
```

```
sudo mv kubectl /usr/local/bin/
```

### Windows

```
PS C:\Windows\system32> Install-Script -Name install-kubectl -Scope CurrentUser -Force                                                                                                                                                          
NuGet provider is required to continue
PowerShellGet requires NuGet provider version '2.8.5.201' or newer to interact with NuGet-based repositories. The NuGet
 provider must be available in 'C:\Program Files\PackageManagement\ProviderAssemblies' or
'C:\Users\jefra\AppData\Local\PackageManagement\ProviderAssemblies'. You can also install the NuGet provider by running
 'Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force'. Do you want PowerShellGet to install and
import the NuGet provider now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
```

### Verification

Verify `kubectl` version 1.15.3 or higher is installed:

```
kubectl version --client
```

> output

```
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T11:13:54Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
```
 
Windows

```
PS C:\Windows\system32> kubectl version --client
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.5", GitCommit:"20c265fef0741dd71a66480e35bd69f18351daea", GitTreeState:"clean", BuildDate:"2019-10-15T19:16:51Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"windows/amd64"}
```

Next: [Provisioning Compute Resources](03-compute-resources.md)
