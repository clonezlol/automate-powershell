# Automações com powershell

---
Este repositório faz parte da sequência de videos lançados pelo canal [Estação da TI](https://youtu.be/ZDmX6lqPThg)

---

# Levantamento de Tags #
O script foi elaborado usando comandos AZ e powershell. Ele é responsável por levantar todos os recursos presentes na subscription para receberem as Tags. O motivo dele existir é facilitar a aplicação de tags em recursos onde as policy da Azure não são efetivas.

**Procedimentos antes de utilizar o script:**

Habilita execução de scripts

```Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser```

Instalado modulo para exportar para o Excel

```Install-Module ImportExcel -AllowClobber -Force```

Instalando modulos az

```Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force```

**Testar comando az graph**

Executar o comando a seguir para validar o download das informaões do graph query 
```
az graph query -q "ResourceContainers | where type=='microsoft.resources/subscriptions/resourcegroups' | project  id, resourceGroup, name, type | union  (Resources | project  id, resourceGroup, name, type)" --subscriptions  <subscription> --first 10 --query "data" -o json | ConvertFrom-Json
```


## Consulta Tags dos Recursos ##
Esta primeira parte do script irá realizar um levantamento de todos os recursos presentes na subscription e salva em um arquivo Excel.

```
$path = "C:\Temp\output.xlsx"
$subscription = <subscription>

$resources = az graph query -q "ResourceContainers | where type=='microsoft.resources/subscriptions/resourcegroups' | project  id, resourceGroup, name, type | union  (Resources | project  id, resourceGroup, name, type)" --subscriptions $subscription --first 1000 --query "data" -o json | ConvertFrom-Json
$outputs = $resources | ForEach-Object {
    [PSCustomObject]@{
        "id" = $_.id
        "ResourceGroup" = $_.resourceGroup
        "ResourceName" = $_.name
        "Type" = $_.type
        "Nome do Ambiente" = ""
        "Tipo do Ambiente" = ""
        "Descricao" = ""
    }
}
$outputs | Export-Excel -workSheetName "Resources" -path $path
```

## Aplica Tags nos Recursos ##
Esta segunda parte do script é responsável por aplicar as tags nos recursos. Este segundo script usa como input o excel gerado pela **consulta de tags**.

```
#O Arquivo de input deve ser o gerado pelo report de tags
$path = "C:\temp\output.xlsx"

Import-Excel -Path $path | ForEach-Object {
    $Tag1 = 'Nome do ambiente=' + $_."Nome do Ambiente"
    $Tag2 = 'Tipo do ambiente=' + $_."Tipo do Ambiente"
    $tag3 = 'Descricao=' + $_.Descricao
    $id = $_.id

    az tag update --resource-id $id --operation merge --tags $Tag1 $Tag2 $tag3
}
```

---
Este repositório faz parte da sequência de videos lançados pelo canal [Estação da TI](https://youtu.be/ZDmX6lqPThg)

---
