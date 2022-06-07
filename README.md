## DOCKER

### exibe informações do ambiente
docker system info
docker info


### listar containers
docker container ls


### excluir container
docker container rm <CONTAINER>


### iniciar / parar container
docker container start <CONTAINER>
docker container stop <CONTAINER>


### listar imagens
docker image ls


### pesquisar imagem
docker search <IMAGE>


### listar redes
docker network ls


### listar volumes
docker volume ls


### download da imagem desejada
docker image pull <IMAGE>


### executar container 
docker container run -dit --name <CONTAINER> --hostname <HOSTNAME_CONTAINER> <IMAGE>
docker container run -dit --name <CONTAINER> -h <HOSTNAME_CONTAINER> <IMAGE>



### conectar o container
docker container attach <CONTAINER>



### desconecta do container sem pará-lo
<CTRL> + <P> + <Q>


### exibir logs 
docker container logs <CONTAINER>



### copiar arquivo para dentro do container
docker container cp <FILE> <CONTAINER>:<DIR_CONTAINER>



### executar comando dentro do container
docker container exec <CONTAINER> <COMMAND>



### acessar e sair do acesso via terminal no docker hub
docker login -u <USER>
docker logout



## IMAGE


### exibe histórico e camadas da imagem
docker image history <IMAGE>


### inspecionar imagem
docker image inspect debian



### criar uma nova imagem a partir das alterações feitas
docker container commit <CONTAINER> <IMAGE>



### salva imagem e empacota para dentro de um arquivo compactado
docker image save <CONTAINER> -o <FILENAME>.tar
du -sh <FILENAME>.tar



### carrega imagem a partir do arquivo
docker image load -i <FILENAME>.tar




## Dockerfile

FROM Distribuição:Versão
COPY Arquivo_Local Caminho_Absoluto_no_Container
RUN Comando
EXPOSE Porta do serviço
CMD Comando executado ao iniciar o Container

## Definições
FROM        Inicializa um novo estágio de compilação e define a imagem de base para instruções subsequentes;
COPY        Copia arquivos ou diretórios de origem local adicionando-os a imagem do container;
ADD         Similar ao parâmetro COPY porém possibilita que a origem seja uma URL bem como a alteração de permissionamento ao adicionar os arquivos a imagem do container;
RUN         Executa os comandos em uma nova camada na parte superior a imagem atual
            É uma boa prática combinar diversos comandos em um unico RUN utilizando de ; e && para a combinação, assim criando apenas uma camada;
EXPOSE      Informa ao docker a porta na qual o container estará escutando enquanto estiver sendo executado
            É possível especificar portas TCP e UDP, caso não seja declarado o tipo de porta, o padrão (TCP) é assumido.
CMD         Só pode existir uma unica instrução deste tipo em um arquivo
            O propósito desta instrução é prover os padrões para a execução do container, podendo ser um executável ou até mesmo opções para o executável definido na instrução ENTRYPOINT
ENTRYPOINT  Possibilita configurar o container para rodar como um executável
            O comando docker run <IMAGE> inicializará o container em seu entrypoint somado ao CMD se existente.



### cria imagem a partir do dockerfile
docker image build -t <CONTAINER> <DOCKERFILE>



### salvar imagem no docker hub
docker login -u <USER_DOCKERHUB>
docker image tag <TAG> <USER_DOCKERHUB>/<IMAGE>
docker image push <USER_DOCKERHUB>/<IMAGE>
docker logout


## VOLUME

Volume é um diretório especialmente designado, seja em um ou mais containers que compartilham o sistema de arquivos UnionFS
Os volumes são projetados para manter os dados, independentemente do ciclo de vida do container.
O Docker nunca exclui um volume automaticamente quando você remove um container.
Por padrão, os volumes criados e gerenciados pelo docker se localizam no diretório /var/lib/docker/volumes

### 3 tipos de Volumes:
Host
Nomeado
Anônimo


### visualizar storage driver em uso
docker system info | grep Storage




## MONTAR VOLUME
É possível montar um volume com os args --volume e -v
Também é possível montar com --mount, porém é mais verboso


### criar volume do tipo host
docker container run -dit --name <CONTAINER> -v <DIR_VOLUME>:/<DIR_VOLUME> <IMAGE>



### criar volume do tipo anônimo
docker container run -dit --name <CONTAINER> -v <VOLUME> <IMAGE>
docker container run -dit --name servidor -v /volume debian



### criar volume do tipo nomeado
docker container run -dit --name <CONTAINER> -v <NAME_VOLUME>:/<DIR_VOLUME> <IMAGE>
docker container run -dit --name servidor -v volume:/volume debian



### criar volume com mount
docker container run -dit --name <CONTAINER> --mount source=<VOLUME>,target=/<VOLUME>  <IMAGE>



### pesquisa dentro da inspeção
docker container inspect <CONTAINER> --format '{{json .Mounts }}' 
docker container inspect <CONTAINER> -f '{{json .Mounts }}'



### pesquisa dentro da inspeção com jq
sudo apt-get update && sudo apt-get install jq -y
docker container inspect <CONTAINER> --format '{{json .Mounts }}' | jq 
docker container inspect <CONTAINER> -f '{{json .Mounts }}' | jq

O mode :z indica que o conteúdo do bind mount é compartilhado entre múltiplos containers
O mode :Z indica que o conteúdo do bind mount é privado e não compartilhado



## RESTAURAR VOLUME


### criar container e seu volume, criar novo container com novo volume e restaurar backup
docker container run -dit -v /<VOLUME> --name <CONTAINER> <IMAGE>
docker container exec <CONTAINER> ls -lR /<VOLUME>
docker container run --rm --volumes-from <CONTAINER> -v $(pwd):/backup alpine ash -c "cd /<VOLUME> && tar xvf /backup/backup.tar --strip 1"
docker container exec <CONTAINER> ls -lR /<VOLUME>



## PLUGINS
docker plugin


### instalar plugin sshfs
docker plugin install vieux/sshfs



## NETWORKING

Por padrão quando um container é criado ele não publica nenhuma de suas portas para o mundo externo.
Para expor porta usar --publish ou -p.



### Recursos de Network utilizados pelo docker:
veth: Virtual Ethernet
bridge: Interface Bridge
iptables: Regras de isolamento de redes



### expor porta ao criar o container
docker container run -dit --name <CONTAINER> -p <PORT_HOST>:<PORT_CONTAINER> <IMAGE>



## DRIVERS

### Tipos
bridge: é o padrão, utilize se precisa executar containers isolados que se comunicam entre si
host: para containers isolados, remove o isolamento de rede entre o container e o host Docker e utiliza a rede diretamente
overlay: redes utilizadas para conectar multiplos Docker Daemons e habilitar os serviços de swarm (cluster) para se comunicarem entre si.
macvlan: redes Macvlan possibilitam que configuremos um endereço MAC a um container, fazendo com que ele apareça como um dispositivo físico na rede.
        O docker daemon faz o roteamento do trafégo entre containers pelo MAC Address.
none: Redes do tipo none disabilitam toda parte de redes do container, não pode ser utilizada com swarm (cluster).
plugins: Utilizados para extender a funcionalidade de redes docker



### cria container com a rede bridge, verifica serviço na porta 80
docker container run -dit --name <CONTAINER> --network bridge -p 80:80 <IMAGE> 
sudo ss -ntpl | grep 80



### lista regras de iptables - para criar o isolamento
sudo iptables -nL




### executar um container com a rede host
### ao utilizar o drive de rede host não é necessário publicar a porta uma vez que não existe uma ponte de conexão
### verificar a stack de rede
docker container run -dit --name <CONTAINER> --network host  <IMAGE>
docker container run -dit --name sem-rede --network none  alpine ash
docker container exec sem-rede ip link show 
docker container exec sem-rede ip route 




## CONECTAR CONTAINERS

### CRIAR CONTAINERS

docker container run -dit --name <CONTAINER> -h <HOSTNAME> <IMAGE>
docker container run -dit --name container1 -h servidor alpine
docker container run -dit --name container2 -h cliente alpine


### Verifique o endereço IP dos containers
docker container exec container1 ip -c a show eth0
docker container exec container2 ip -c a show eth0


### TESTE DE CONECTIVIDADE POR IP
docker container exec container1 ping -c4 172.17.0.3
docker container exec container2 ping -c4 172.17.0.2


### TESTE DE CONECTIVIDADE POR NOME
docker container exec container1 ping -c4 cliente
docker container exec container1 ping -c4 servidor
não funciona, pinga apenas para o container interno
