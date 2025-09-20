![LVMM logo](https://lvmm.mx/wp-content/uploads/2019/05/LVMM_logo_500-90x90.png)

# Manual de uso del clúster LVMM

<a name="ssh"></a>
## 1. ¿Cómo acceder al cluster LVMM?

La coneción al clúster computacional del Departamento de Modelación de Nanomateriales es por medio del protocolo [OpenSSH](https://www.openssh.com/) (Open Secure Shell, por sus siglas en inglés).

Dentro de la red del CNyN ésto se realiza por medio del siguiente comando:

```bash
$ ssh usuario@192.168.100.237
```

donde `usuario` se refiere al nombre del usuario al que se quiere acceder en el clúster, con éste comando nos preguntará nuestra clave:

```bash
By accessing this system, you consent to the following conditions:
- This system is for authorized use only.
- Any or all uses of this system and all files on this system may be monitored.
- Communications using, or data stored on, this system are not private.

usuario@192.168.100.237's password:

```

Una vez que logrando igresar al clúster LVMM

```bash
usuario@lvmm:~$
```


<a name="sftp"></a>
## 2. ¿Cómo transferir información al clúster LVMM?

Para transferir información entre el cluster se utiliza el mismo protocolo `SSH`, ya sea con `sftp` o `scp`.

`sftp` se refiere a la implementación de un servicio FTP (File Transfer Protocol, por sus siglas en inglés) sobre el protocolo SSH:

```bash
$ sftp usuario@192.168.100.237
```

otra forma de transferir archivos puede ser con la herramienta `scp`, copiar sobre el protocolo SSH, la forma de usarlo es:

> **scp** [opciones] \<**origen**> \<**destino**>

el \<**origen**> ó \<**destino**> remoto sería:

> usuario@192.168.100.237:\<**ruta al archivo/directorio**>

sí es un directorio no olvide incluir la opción **-r** para que el copiado sea recursivo, por ejemplo para copiar archivos remotos a la PC **localhost**:

```bash
localhost:resultados $ scp -r usuario@192.168.100.237:/tmpu/grupo/usuario/datos .
```

<a name="module"></a>
# 3. ¿Cómo utilizar los programas instalados en el clúster LVMM?

Para poder utilizar los programas ya instalabos en el cúster LVMM es necesario cargar el módulo adecuado, ([Environment Modules](https://modules.sourceforge.net/)).

<a name="ml_av"></a>
Para ver la lista de aplicaciones, librerías y utilidades, se pueden consultar con el siguiente comando:

```bash
$ module available
```

o de forma adbreviada:

```bash
$ ml av
```

```bash
----------------------------------- /share/modules/Applications ------------------------------------
abinit/10.0.7.1-intel    gromacs/2024.1-gnu-mkl     vasp/6.4.1-intel           
amber/20-Update12        irrep/1.1                  vasp/6.4.1-intel-vtst         
autodock/4.2.6-intel     lammps/2Aug2023-intel      vasp/6.4.1-MD-intel           
boltztrap/20.7.1         namd/2.14-intel            vasp/6.4.1-nvhpc-gpu            
cp2k/2024.1-gnu-mkl      py4vasp/0.5.1              wannier90/2.1.0-intel
critic2/1.1.git-intel    qe/7.1-intel               wannier90/3.1.0-intel         
crystal17/1.0.2-intel    siesta/honpas-4.1.5-intel  wannier90/3.1.0-serial-intel  
dftb+/24.1-intel         tinker/8.10.2-intel        wien2k/23.2-intel             
gromacs/2024.1-gnu-cuda  

Key:
default-version  modulepath  
```


<a name="ml_load"></a>
Si queremos utilizar, por ejemplo, el programa VASP para GPU, debemos primero cargar el módulo `vasp/6.4.1-nvhpc-gpu` con el siguiente comando:

```bash
$ ml load vasp/6.4.1-nvhpc-gpu
```


```bash
Loading vasp version 6.4.1
Loading mkl version 2023.0.0
Loading tbb version 2021.8.0
Loading compiler-rt version 2023.0.0
Loading fftw version 3.3.10
Loading hdf5 version 1.14.3

Loading vasp/6.4.1-nvhpc-gpu
  Loading requirement: nvhpc/24.5 tbb/latest compiler-rt/latest mkl/latest fftw/3.3.10-nvhpc
    hdf5/1.14.3-nvhpc
```

Los mensajes de salida indican que también se han cargado las dependencias requeridas para que funcione correctamente nuestra aplicación, una vez realizado ésto podémos correr `vasp_std`, `vasp_ncl` o `vasp_gam`.

```bash
$ mpirun -n 4 vasp_ncl
 running    4 mpi-ranks, with    2 threads/rank, on    1 nodes
 distrk:  each k-point on    4 cores,    1 groups
 distr:  one band on    1 cores,    4 groups
 vasp.6.4.1 05Apr23 (build Jul  1 2024 01:14:50) complex                         
 ```

con éste comando se ejecuta el programa `vasp_ncl` en el **nodo maestro**, lo que **NO es recomendado**).

<a name="slurm"></a>
# 4. ¿Cómo correr una aplicación en el LVMM?

El clúster HPC LVMM cuenta con el sistema de administración de recursos [SLURM](https://slurm.schedmd.com) (Simple Linux Utility for Resource Management, por sus siglas en inglés).

<a name="sinfo"></a>
## 4.1 sinfo

El comando `sinfo` nos ayuda a obtener información sobre el cluster,

```shell
$ sinfo
```

```shell
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
q_gpu        up   infinite      2   idle compute-0-[0,5]
q_mem        up   infinite      1    mix compute-0-1
q_cpu*       up   infinite      3    mix compute-0-[1-2,4]
q_cpu*       up   infinite      1  alloc compute-0-3
```
éste comando nos muestra las **particiones** (colar) de ejecución de trabajos, podemos ver que ésta instalación de `SLURM` cuenta con 3 particiones distintas: `gpu`, `mem` y `cpu`, también podemos ver el éstado de los nodos de éstas particiones; La partición `gpu` se encuentra **desocupada** y que cuenta de los nodos de cómputo: `compute-0-0` y `compute-0-5`; también podemos ver que de la partición `cpu`, el nodo `compute-0-3` se encuentra **ocupado** y los nodos `compute-0-1`, `compute-0-2` y el nodo `compute-0-4` se encuentran en un éstado de ocupación **mixto**.

Podemos obtener la misma información pero ahora centrada en los nodos en lugar de las particiones con la opción `--Node` o en la forma abreviada `--N`,

```bash
$ sinfo --Node
NODELIST     NODES PARTITION STATE
compute-0-0      1     q_gpu idle  
compute-0-1      1    q_cpu* mix   
compute-0-1      1     q_mem mix   
compute-0-2      1    q_cpu* mix   
compute-0-3      1    q_cpu* alloc
compute-0-4      1    q_cpu* mix   
compute-0-5      1     q_gpu idle  
```


<a name="srun"></a>
## 4.2 srun

Para correr `vasp` en el cluster se puede ejecutar con la instrucción `srun` de la siguiente manera:

```bash
$ srun --partition mem --ntasks 16 vasp_ncl
```
```bash
 running   16 mpi-ranks, on    1 nodes
 distrk:  each k-point on   16 cores,    1 groups
 distr:  one band on    4 cores,    4 groups
 vasp.6.4.1 05Apr23 (build Feb 23 2024 02:44:29) complex                        
```

con ésto corremos el programa `vasp_ncl` en la partición `mem` con 15 tarea, el inconveniente de ésta forma de correr el programa es que tenemos que esperar hasta que estén disponibles los recursos requeridos y durante su ejecución.


<a name="sbatch"></a>
## 4.3 sbatch

La forma idonea para mandar un trabajo a una cola de ejecución es por medio del comando `sbatch`, para ésto tenemos que gerenar un script:

`vasp.job:`
```shell
#!/usr/bin/env bash
srun vasp_ncl
```
una vez creado el script se somete el script con el comando `sbatch`de la sigueinte forma:

```shell
$ sbatch --ntasks=16 --nodes=1 --partition=q_cpu vasp.job
```
```
Submitted batch job 176
```
ésto nos regresa un número `176` el cual es el `ID` del trabajo.

podemos agregar las opciones de la linea de comando de `sbatch` al mismo script, junto con otras opciones, de la siguiente forma:

`vasp.job:`
```shell
#!/bin/bash
#SBATCH --job-name=VASP_test        # Nombre para identificar el trabajo -J
#SBATCH --nodes=1                   # utilizar sólo un nodo -N
#SBATCH --ntasks=16                 # ejecitar 16 tareas -n
#SBATCH --cpus-per-task=1           # CPUs por tarea
#SBATCH --ntasks-per-node=16        # tareas por nodo
#SBATCH --output vasp_test-%j.o     # el nombre del archivo a donde escribir las salidas de la ejecución
#SBATCH --error  vasp_test-%j.e     # nombre del archivo para escribir los errores de ejecución
# %j se refiere al ID asignado a la hora de someter el script
#SBATCH --partition=q_cpu           # partición en la cual ejecutar el script -p

source /etc/profile.d/modules.sh   # Inicializar el sistema de módulos
ml vasp/6.4.1-intel                # cargar el módulo adecuado

srun vasp_ncl                      # ejecutar vast_ncl en los recursos solicitados
```
para someter éste último script sólo tenemos que mandar como parametro el nombre del script (`vasp.job`) a `sbatch`:
```bash
$ sbatch vasp.job
```
```shell
Submitted batch job 177
```


<a name="squeue"></a>
## 4.4 squeue

Para ver el éstado de ejecución de los trabajos en la cola podemos utilizar la instrucción `squeue`.

```bash
$ squeue
```
```bash
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             164       q_cpu  model_4   jperez  R 16-10:19:13     1 compute-0-3
             174       q_cpu    model   jperez  R 4-07:57:35      1 compute-0-2
             173       q_cpu     VASP vgalingo  R 2-12:45:11      1 compute-0-1
             175       q_cpu     Test  wmedina  R   22:59:14      1 compute-0-2
             171       q_cpu      aex  wmedina  R 1-20:57:02      1 compute-0-3
             172       q_cpu   10-35s   clfdez  R    7:05:41      1 compute-0-1
             177       q_cpu VASP_tes    suser  R      16:32      1 compute-0-1
             178       q_cpu model_dm  emarmol  R    6:06:50      1 compute-0-3
             182       q_cpu model_fm vgalindo  R    5:47:16      1 compute-0-4
             184       q_cpu      igc  ncampos  R      56:32      1 compute-0-1
             186       q_mem Na2SO2_d    jvzmg  R    7:05:41      1 compute-0-1
```
podemor verificar que nuestro trabajo esta corriendo (R), podemos ver sólo nuestros trabajos con la opción `--user $USER`

```bash
$ squeue --user $USER
```
```bash
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             177       q_cpu VASP_tes    suser  R      18:22      1 compute-0-1
```


<a name="scancel"></a>
## 4.5 scancel
se puede cancelar el trabajo con el comando `scancel`:
```bash
$ scancel 177
```
```bash
$ squeue -u $USER
```
```bash
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             177         cpu VASP_tes    suser  CA      18:22      1 compute-0-1
```
podemos ver que el trabajo cambió de estado a cancelado (CA). También podemos utilizar el nombre el trabajo `VASP_test` con la opción `--name`,
```bash
$ scancel --name VASP_test
```


<a name="salloc"></a>
## 4.6 salloc

También podemos ejecutar nuestros programas de una forma un poco más interactiva utilizando el comando `salloc`, el cual nos aloja los recursos solicitados y nos da un shell ineractivo

```bash
$ salloc -p q_cpu -N 1 -n 16 -J VASP_test -t 60
```
```bash
salloc: Granted job allocation 178
$
```
una vez terminada nuestra ejecución o pruebas nos podemos salir como en cualquier shell,
```bash
$ exit
```
ó

```bash
$ logout
```

o también se puede utilizar el acceso rápido `ctrl + d`


---
# 5. Referencias:

- The SLURM website: https://slurm.schedmd.com/

- The SLURM documentation: https://slurm.schedmd.com/documentation.html

- The SLURM user community: https://groups.google.com/g/slurm-users

Aldo Rodríguez Guerrero, 2024
