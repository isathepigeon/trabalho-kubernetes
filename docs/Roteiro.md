## Contexto

A Web Solutions e uma empresa ficticia que assim como meu empregador atual utiliza-se de Virtual Machines. Porem as VMS sao maquinas inteiras rodando em servidores fisicos da Microsoft e pro caso da Web Solutions que so precisa compartilhar um aplicativo + seu environment, faz mais sentido fazer deployment de containers do que de VMs inteiras.

Os Kubernetes entao vao ser esse servico com uma serie de componentes que dialogam entre si pra permitir o acesso continuo a uma determinada aplicacao.

Voce tem o cluster que e como se fosse uma fabrica, os nodes que sao como se fossem galpoes e vao se dividir entre worker  nodes e um master node e os pods que sao caixas contendo os produtos que sao os containers. 

O pod e agendado pra um determinado Node uma unica vez.

Como?

A grande parte do Kubernetes que permite que ele seja mais reliable do que o Docker inicialmente era, e que voce escreve um objetivo final atraves de um config map e o kubernetes vai ficar comparando o estado atual da aplicacao com o objetivo final continuamente e fazendo os ajustes necessarios, como por exemplo destruir e recriar um pod caso isso seja necessario pra manter o objetivo final q foi definido no configuration map.

Um node inteiro pode tambem ser destruido em caso de falha de rede ou de hardware.

Como proof of concept, eu coloquei dois servidores web — Nginx e Apache — rodando de forma independente no mesmo cluster."

Quando eu digo de forma independente eu falo em relacao ao service entao as portas estao separadas, o deployment e os labels estao separados mas como eu usei aqui o minikube na realidade vai ser um node so. Se a gente tivesse em um ambiente de multiplos nodes normalmente o scheduler espalha entre nodes diferentes por uma questao logica de que um node pode cair por uma falha de rede ou de hardware e ai o kubernetes pode substituir esse worker node.


Numa situacao real cada replica vai em um node justamente por isso.

## O que é YAML 

E a nossa configuration map language de escolha aqui foi o YAML.

Esses arquivos no Kubernetes sao chamados de manifestos, seguem quatro campos principais: apiVersion, kind, metadata e spec. Eu descrevo como meu ambiente deve ficar e o sistema trabalha continuamente pra manter esse estado. Se um pod cai, ele sobe outro sozinho. Se eu preciso dobrar a capacidade horizontalmente e so mudar replicass de 2 pra 4 nesse caso por exemplo.

O YAML e a forma mais human readable de voce definir a configuracao do seu container e ele define principalmente por identacao o que vai dentro do que.
A estrutura vai ter key value pairs como por exemplo aqui eu usei replicas: 2, esses pares vao ser marcados por hifen e dentro ou fora vai ser definido pela identacao ou seja o espaco entre o comeco e o texto em si
e tem que ser sempre espaco e nunca tab porque um espaco fora do lugar muda completamente o significado do arquivo


---

## Demo
Criei dois Dockerfiles com imagens oficiais leves, `nginx:alpine` e `httpd:alpine`, cada um copiando uma página HTML personalizada.

O que isso significa
Eu na minha maquina pessoal utilizo a distribuicao Ubuntu do Linux. O Linux nada mais e que um Kernel e distributions sao combinacoes diferentes de aplicacoes em volta do mesmo Kernel. Essas aplicacoes que fazem system requests pro kernel que e o unico programa capaz de se comunicar diretamente com o hardware.
Voce tem o Debian que e a distro mais antiga, a Ubuntu que e user friendly, a Fedora e aqui pro minikube eu optei pela Alpine porque ela pesa somente 5MB ela e muito usada em containers justamente por ser muito leve.

o Nginx e o Apache sao aplicacoes que funcionam como webservers e vao ficar fazendo o trafego de informacoes e o load balancing, o Apache inclusive eh httpd onde o D eh de Deamon eh o nome que se da pra aplicacoes que trabalham no background sem interacao do usuario.

Fiz o build das imagens `webso-nginx` e `webso-apache` dentro do ambiente do Minikube.
![[Pasted image 20260728112356.png]]

FROM httpd:alpine
COPY index.html /usr/local/apache2/htdocs/index.html

![[Pasted image 20260728112601.png]]

![[Pasted image 20260728112631.png]]


Depois escrevi quatro manifestos: um Deployment e um Service pro Nginx, um deployment e um Service pro Apache. Os Deployments declaram duas réplicas de cada aplicação na parte do key value pair spec: 2. Os Services são do tipo NodePort: o Nginx exposto na porta 8080 e o Apache na 8081, ca gente ta fazendo aqui um portfoward pra dizer onde minha aplicacao isinha a braba deve ser exposta ,em qual porta mas eh importante salientar que isso nao eh igual a um portfowarding de networking quando voce tem um doublenat, aqui e so o nome que se da ao ato de especificar em qual porta voce vai encontrar qual servicoe. entao aqui o 8080 eh o Apache rosa choque com brilhinhos, e o Nginx roxinho com brilhinhos na porta 8081.

---

## Conclusão

Concluindo: o YAML funciona como uma linguagem de contrato entre o programador e os containers e essa solucao eh muito mais reliant, stable e scalable do que uma VM inteira. Obrigada!"
