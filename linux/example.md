# 脚本案例

## 前言
这是我写的一个简单的自动化构建docker服务的一个样例，涉及配置文件修改，生成相应的配置文件等等，留作参考

```
#!/bin/bash
#节点数量
NUM=5
#选择网络 172.xx.xx.0 10.xx.xx.0
NETWORK=172.30.0.0
#color
RED='\033[1;31m'
GRE='\033[1;32m'
NOC='\033[0m'
WORKDIR=$(pwd)
#生成私钥对
./chain33-cli valnode  init_keyfile -n $NUM
#修改配置文件
echo -e "${GRE} start auto modify config ${NOC}"
validatorNodes=""
nodes=""
array_nodes=()
prefix=$(echo ${NETWORK%?})
for((i=1;i<=$NUM;i++));
do
validatorNodes=${validatorNodes},\""${prefix}$[$i+1]:46656"\"
array_nodes[$[$i-1]]=${prefix}$[$i+1]
done
#echo -e "${GRE} IP ${array_nodes[*]} ${NOC}"
#删掉第一个逗号
validatorNodes=$(echo $validatorNodes|sed '/^,/s///g')
validatorNodes=validatorNodes=[$validatorNodes]
# echo -e "${GRE} $validatorNodes ${NOC}"
sed -i "s/^validatorNodes=.*/${validatorNodes}/g" chain33.toml
rm -rf ${prefix}*
version=$(./chain33 -v)
echo -e "${GRE} chain33 version:$version ${NOC}"
for((i=0;i<$NUM;i++));
do
mkdir -p ${array_nodes[$i]}/datadir
cp priv_validator_$i.json  ${array_nodes[$i]}/priv_validator.json
cp genesis_file.json  ${array_nodes[$i]}/genesis.json
cp chain33.toml   ${array_nodes[$i]}/chain33.toml
done
#生成docker-compose.yaml文件
rm -rf docker-compose.yaml
CONTEXT=""
for ip in ${array_nodes[*]}
do
CONTEXT=${CONTEXT}"
  chain33-${ip}:
    image: chain33:${version}
    container_name: chain33-${ip} # 容器服务名
# 暴露8801端口
    expose:
      - 8801
      - 46656
    networks:
      app_net:
        ipv4_address: ${ip}
    volumes:
      - \"${WORKDIR}/${ip}/datadir:/app/datadir/\"
      - \"${WORKDIR}/${ip}/chain33.toml:/app/chain33.toml\"
      - \"${WORKDIR}/${ip}/genesis.json:/app/genesis.json\"
      - \"${WORKDIR}/${ip}/priv_validator.json:/app/priv_validator.json\"
"
done
#写入docker-compose.yaml文件
cat > docker-compose.yaml << EOF
version: "3"

services:
   ${CONTEXT}
networks:
  app_net:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: ${NETWORK}/24
EOF
cat > Dockerfile << EOF
FROM ubuntu:latest
WORKDIR /app
COPY chain33 chain33
COPY chain33-cli chain33-cli
ENTRYPOINT ["/app/chain33","-f","/app/chain33.toml"]
EOF
echo -e "${GRE} docker-compose.yaml,Dockerfile file genersis sucess! ${NOC}"
echo -e "${GRE} start build chain33 image ${NOC}"
docker images |grep  chain33 | awk '{print $3}'| xargs docker rmi -f
docker ps -a |grep chain33-${prefix}| awk '{print $14}'| xargs docker rm
docker build . -f Dockerfile -t  chain33:${version}
if [ $? -ne 0 ]
then
  echo -e "${RED} docker build chain33 image failed! ${NOC}"
else
  echo -e "${GRE} docker build chain33 image sucess! ${NOC}"
fi
```
