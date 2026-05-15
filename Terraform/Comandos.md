# Sistemas de Arquivos
## 1. Arquivos de código (.tf)
```txt
projeto/
├── main.tf
├── providers.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
```
Aqui é onde fazemos a receita de tudo que queremos criar.
### 1.1 Variables
No Arquivo `variables.tf`:
```hcl
variable "bucket_name" {
  type = string
}
```
E para usar no arquivo `main.tf`:
```hcl
module "s3" {
  source = "./modules/s3_bucket"
  bucket_name = var.bucket_name
}
```

### 1.2 terraforms.tfvars
Ele é onde colocamos os **valores reais das variaveis**.
```hcl
bucket_name = "nome do bucket que disse que existia em variables.tf (tópico 1.1)"
```

### 1.3 Outputs
No arquivo `outputs.tf`
```hcl
output "bucket_id" {
  value = aws_s3_bucket.this.id
}
```
E No arquivo `main.tf` podemos usar:
```hcl
output "s3_bucket_id" {
  value = module.s3.bucket_id
}
```

## 2. Cache interno (.terraform)
```txt
projeto/
├── .terraform/
│   ├── providers/
│   ├── modules/
│   └── plugins/
```
Aqui é onde ficam os providers baixados (AWS, Azure, GCP, etc, ...), plugins e outros modulos. É como se fosse um `pip` do python.

## 3. Estado readl da infra (terraform.tfstate)
Arquivo mais importante do terraform, não mexa manualmente nele.
```txt
meu-projeto/
├── terraform.tfstate
├── terraform.tfstate.backup
```
Ele contém o que realmente foi criado na **cloud**, IDs reais e o estado atual da infra.

## 4. Modules
```txt
project/
├── main.tf
├── variables.tf
├── outputs.tf
├── locals.tf
│
└── modules/
    └── s3_bucket/
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
```
Um `module` é um *mini terraform isolado*, é uma **pasta reutilizável de infraestrutura**

### 4.1 Como usar o module
No `main.tf`
```hcl
module "s3" {
  source = "./modules/s3_bucket"
  bucket_name = var.bucket_name
}
```

## 5. Locals
Arquivos `locals.tf`
```hcl
locals {
  environment = "dev"
  project     = "data-lake"
}
```
Podemos usar no `main.tf`:
```hcl
resource "aws_s3_bucket" "this" {
  bucket = "${local.project}-${local.environment}-bucket"
}
```
Quando usar `locals.tf`:
- Queremos evitar repetição;
- Queremos montar strings;
- Queremos derivar valores.

# Comandos Terraforms
- *funções:* manipulam dados (`merge`, `concat`, etc)
- *meta-arguments:* controlam recursos (`for_each`)

## `for_each`
```hcl
# for_each: cria múltiplos recursos dinamicamente
variable "users" {
    default = ["alice", "bob", "charlie"]
}

resource "aws_iam_user" "users" {
    for_each = toset(var.users)
    name = each.key
}
```

## `count`
```hcl
# count: mais simples que for_each (menos flexível)
resource "aws_instance" "server" {
    count = 2
    ami = "ami-123"
    instance_type = "t2.micro"
    tags = {
        Name = "server-${count.index}"
    }
}
```

## `lookup`
```hcl
# lookup: busca valor em um map com fallback, é como um get se não existir devolvemos um valor default.
variable "instance_types" {
  default = {
    dev  = "t2.micro"
    prod = "t3.large"
  }
}

locals {
  instance_type = lookup(var.instance_types, "dev", "t2.nano")
}
```

## `merge`
``` hcl
# merge: junta mútiplos maps
locals {
  default_tags = {
    Owner = "Luiz"
  }

  extra_tags = {
    Env = "dev"
  }

  tags = merge(local.default_tags, local.extra_tags)
}
```

## `concat`
```hcl
# concat: junta listas
locals {
  list1 = ["a", "b"]
  list2 = ["c", "d"]

  result = concat(local.list1, local.list2)
}
```

## `coalesce`
``` hcl
# coalesce: retorna o primeiro valor não nulo
locals {
  name = coalesce(null, "", "default-name")
}
```

## `element`
``` hcl
# element: pega elemento por indice
locals {
  zones = ["us-east-1a", "us-east-1b"]

  zone = element(local.zones, 1)
}
```

## `file`
``` hcl
# file: lê o conteúdo de um arquivo
locals {
  script = file("${path.module}/script.sh")
}
```

## `join`
```hcl
# join: junta lista em string
locals {
  names = ["alice", "bob"]

  result = join(", ", local.names)
}
```

## `split`
```hcl
# string: divide string em lista
locals {
  data = "a,b,c"

  result = split(",", local.data)
}
```

## `lenght`
```hcl
# lenght: devolve o tamanho de uma lista/map/string
locals {
  size = length(["a", "b", "c"])
}
```

## `lower` / `upper`
```hcl
locals {
  lower = lower("HELLO")  # fica minusculo
  upper = upper("hello")  # fica maiusculo
}
```

## `replace`
```hcl
# replace: substitui texto
locals {
  result = replace("hello world", "world", "terraform")
}
```

## `toset`
```hcl
# toset: remove duplicadas (e vira um set)
locals {
  result = toset(["a", "a", "b"])
}
```

## `tolist`
```hcl
# tolist: faz virar uma lista
locals {
  result = tolist(toset(["a", "b"]))
}
```

## `tomap`
```hcl
# tomap: é como um dicionario/hashmap
locals {
  result = tomap({
    a = 1
    b = 2
  })
}
```

## `jsonencode`
```hcl
# jsonencode: converte para json
locals {
  json = jsonencode({
    name = "luiz"
    age  = 30
  })
}
```

## `jsondecode`
```hcl
# jsondecode: transforma uma string em um map
locals {
  obj = jsondecode("{\"name\":\"luiz\"}")
}
```

## `$`
```hcl
# $: interpola strings
variable "ambiente" {
  default = "producao"
}

resource "aws_instance" "web" {
  # O nome da instância será "servidor-web-producao"
  tags = {
    Name = "servidor-web-${var.ambiente}"
  }
}
```