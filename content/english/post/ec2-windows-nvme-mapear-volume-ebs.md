---
categories:
- CloudOps
date: 2026-02-28
description: Aprenda como identificar corretamente volumes EBS em
  instâncias EC2 Windows.
draft: false
slug: ec2-windows-nvme-mapear-volume-ebs
tags:
- AWS
- EC2
- Windows Server
- EBS
- NVMe
- CloudOps
title: "Como mapear corretamente Volume ID do EBS em uma EC2 Windows"
---

E aew pessoal,

Recebi uma solicitação para aumentar o tamanho de uma unidade específica em um servidor Windows hospedado na AWS.

Até aqui, algo comum.

O problema?  
A instância EC2 possuía **múltiplos volumes EBS anexados**.

E quando há vários discos no mesmo servidor, existe um risco real:

> ❗ Aumentar o volume errado na AWS.

Em ambientes produtivos (principalmente banco de dados ou SAP), isso pode gerar impacto significativo.




------------------------------------------------------------------------
## Método 1 --- Utilizando o utilitário ebsnvme-id

A AWS disponibiliza o utilitário `ebsnvme-id` para mapear discos NVMe
aos volumes EBS.

Esse comando permite identificar:

-   Número do disco NVMe
-   ID do volume EBS
-   Nome do dispositivo configurado na instância

Executar:

``` bash
ebsnvme-id.exe
```

Ou para um disco específico:

``` bash
ebsnvme-id.exe /dev/nvme1n1
```

Exemplo de retorno:
  ![Mapeamento EBS NVMe](/images/ebs.png)

------------------------------------------------------------------------

##  Método 2 --- Identificação via PowerShell

O `SerialNumber` do disco contém o Volume ID do EBS.

Execute no PowerShell como Administrador:

``` powershell
Get-Disk | ForEach-Object {
    $disk = $_
    Get-Partition -DiskNumber $disk.Number | ForEach-Object {
        [PSCustomObject]@{
            DiskNumber   = $disk.Number
            DriveLetter  = $_.DriveLetter
            SizeGB       = [math]::Round($_.Size/1GB,2)
            VolumeID     = ($disk.SerialNumber -replace '_00000001','')
        }
    }
}
```

### Exemplo de saída:

    DiskNumber DriveLetter SizeGB VolumeID
    ---------- ----------- ------ ------------------------
    0          C           120    vol-052ddc735fba28ced
    6          E           16384  vol-05c810fda39b84d84
    4          F           5548   vol-09748167b6350a54d

O campo `VolumeID` corresponde exatamente ao ID do volume no console da
AWS.

------------------------------------------------------------------------

## Boas práticas operacionais

Antes de qualquer alteração em disco:

-   Validar Volume ID
-   Validar tamanho provisionado (GB)
-   Confirmar que está de fato na instância correta
-   Registrar evidência técnica no ticket

Nunca utilizar apenas o tamanho ou a ordem dos discos como referência.

------------------------------------------------------------------------

Até a próxima e keep learning =)
