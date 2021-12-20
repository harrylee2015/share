# Fabric插件配置

上一节已经介绍了[fabric1.4开发环境如何搭建](fabric-1.4环境搭建)

本节主要介绍fabric跨链插件的如何配置使用,在此之前我们应该先完成[Sidecar节点信息配置](Sidecar使用文档)

## fabric插件编译

```
git clone  https://gitlab.33.cn/link33/sidecar-client-fabric

cd  sidecar-client-fabric

make  fabric1.4

```

## 将插件及配置拷贝到plugins

```
//装到sidecar-client-fabric项目路径下

mkdir -p $SIDECAR_PATH/plugins/fabric

cp  ./build/fabric-client-1.4   $SIDECAR_PATH/plugins/
cp ./config     $SIDECAR_PATH/plugins/fabric

```
插件的主要配置文件有

```

├── fabric
│   ├── config.yaml
|   └── crypto-config/
│   └── fabric.toml
│   └── fabric.validators


```

* Fabric网络配置

```

// 复制你所部署的Fabric所产生的crypto-config文件夹
cp -r /home/harry/fabric/fabric-samples/first-network/crypto-config  $SIDECAR_PATH/plugins/fabric/


// 复制Fabric上验证人证书
cp $SIDECAR_PATH/fabric/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/msp/signcerts/peer1.org2.example.com-cert.pem  $SIDECAR_PATH/fabric/fabric.validators

```

* 修改config.yaml配置文件

config.yaml文件记录的Fabric网络配置，需要使用绝对路径，把所有的路径都修改为 crypto-config文件夹所在的绝对路径。

```

path: {CONFIG_PATH}/fabric/crypto-config => path: $SIDECAR_PATH/plugins/fabric/crypto-config

```

同时需要修改所有的Fabric 的IP地址，在[fabric1.4环境搭建](Fabric 1.4环境搭建)一节,我们介绍了如何在本地通过域名地址的方式访问fabric docker容器环境，这里我们可以把访问地址全部改成相应的域名地址，如：

```
url: grpcs://localhost:7050 => url: grpcs://orderer.example.com:7050

```

下面给出一份配置config.yaml配置文件示例:

```
version: 1.0.0

client:

  # Which organization does this application instance belong to? The value must be the name of an org
  # defined under "organizations"
  organization: org2

  logging:
    level: info


  # Root of the MSP directories with keys and certs.
  cryptoconfig:
    path: /home/harry/sidecar/plugins/fabric/crypto-config

  # Some SDKs support pluggable KV stores, the properties under "credentialStore"
  # are implementation specific
  credentialStore:
    # [Optional]. Used by user store. Not needed if all credentials are embedded in configuration
    # and enrollments are performed elswhere.
    path: "/tmp/state-store"

    # [Optional]. Specific to the CryptoSuite implementation used by GO SDK. Software-based implementations
    # requiring a key store. PKCS#11 based implementations does not.
    cryptoStore:
      # Specific to the underlying KeyValueStore that backs the crypto key store.
      path: /tmp/msp

    # BCCSP config for the client. Used by GO SDK.
  BCCSP:
    security:
      enabled: true
      default:
        provider: "SW"
      hashAlgorithm: "SHA2"
      softVerify: true
      level: 256

  tlsCerts:
    #[Optional]. Use system certificate pool when connecting to peers, orderers (for negotiating TLS) Default: false
    systemCertPool: true

    #[Optional]. Client key and cert for TLS handshake with peers and orderers
    client:
      key:
        path: /home/harry/sidecar/plugins/fabric/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/client.key
      cert:
        path: /home/harry/distribute_sidecar/sidecar04/plugins/fabric/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/client.crt

#
# [Optional]. But most apps would have this section so that channel objects can be constructed
# based on the content below. If an app is creating channels, then it likely will not need this
# section.
#
channels:

  #[Required if _default not defined; Optional if _default defined].
  # name of the channel
  mychannel:

    # list of orderers designated by the application to use for transactions on this
    # channel. This list can be a result of access control ("FBI" can only access "ordererA"), or
    # operational decisions to share loads from applications among the orderers.  The values must
    # be "names" of orgs defined under "organizations/p unable to load config backend: loading config feers"
    # deprecated: not recommended, to override any orderer configuration items, entity matchers should be used.
    #    orderers:
    #      - orderer.citizens.com

    #[Required if _default peers not defined; Optional if _default peers defined].
    # list of peers from participating orgs
    peers:
      peer1.org2.example.com:
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true
      # peer0.org2.example.com:
      #   endorsingPeer: true
      #   chaincodeQuery: true
      #   ledgerQuery: true
      #   eventSource: true
#
# list of participating organizations in this network
#
organizations:
  org1:
    mspid: Org1MSP

    # This org's MSP store (absolute path or relative to client.cryptoconfig)
    cryptoPath:  peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

    peers:
    - peer0.org1.example.com
    - peer1.org1.example.com

    # [Optional]. Certificate Authorities issue certificates for identification purposes in a Fab based
    # network. Typically certificates provisioning is done in a separate process outside of the
    # runtime network. CA is a special certificate authority that provides a REST APIs for
    # dynamic certificate management (enroll, revoke, re-enroll). The following section is only for
    # CA servers.
    # certificateAuthorities:
    #   - ca.fbi.citizens.com

  # the profile will contain public information about organizations other than the one it belongs to.
  # These are necessary information to make transaction lifecycles work, including MSP IDs and
  # peers with a public URL to send transaction proposals. The file will not contain private
  # information reserved for members of the organization, such as admin key and certificate,
  # ca registrar enroll ID and secret, etc.

  org2:
    mspid: Org2MSP

    # This org's MSP store (absolute path or relative to client.cryptoconfig)
    cryptoPath:  peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp

    peers:
    - peer0.org2.example.com
    - peer1.org2.example.com

#
# List of orderers to send transaction and channel create/update requests to. For the time
# being only one orderer is needed. If more than one is defined, which one get used by the
# SDK is implementation specific. Consult each SDK's documentation for its handling of orderers.
#
orderers:
  orderer.example.com:
    url: grpcs://orderer.example.com:7050

    # these are standard properties defined by the gRPC library
    # they will be passed in as-is to gRPC client constructor
    grpcOptions:
      ssl-target-name-override: orderer.example.com
      # These parameters should be set in coordination with the keepalive policy on the server,
      # as incompatible settings can result in closing of connection.
      # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
      allow-insecure: false

    tlsCACerts:
      # Certificate location absolute path
      path: /home/harry/sidecar/plugins/fabric/crypto-config/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem

#
# List of peers to send various requests to, including endorsement, query
# and event listener registration.
#
peers:
  peer0.org1.example.com:
    # this URL is used to send endorsement and query requests
    url: grpcs://peer0.org1.example.com:7051
    eventUrl: grpcs://peer0.org1.example.com:7053
    grpcOptions:
      ssl-target-name-override: peer0.org1.example.com
      # These parameters should be set in coordination with the keepalive policy on the server,
      # as incompatible settings can result in closing of connection.
      # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
      allow-insecure: false

    tlsCACerts:
      # Certificate location absolute path
      path: /home/harry/sidecar/plugins/fabric/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem

  peer1.org1.example.com:
    # this URL is used to send endorsement and query requests
    url: grpcs://peer1.org1.example.com:8051
    eventUrl: grpcs://peer1.org1.example.com:8053
    grpcOptions:
      ssl-target-name-override: peer1.org1.example.com
      # These parameters should be set in coordination with the keepalive policy on the server,
      # as incompatible settings can result in closing of connection.
      # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
      allow-insecure: false

    tlsCACerts:
      # Certificate location absolute path
      path: /home/harry/sidecar/plugins/fabric/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem

  peer0.org2.example.com:
    # this URL is used to send endorsement and query requests
    url: grpcs://peer0.org2.example.com:9051
    eventUrl: grpcs://peer0.org2.example.com:9053
    grpcOptions:
      ssl-target-name-override: peer0.org2.example.com
      # These parameters should be set in coordination with the keepalive policy on the server,
      # as incompatible settings can result in closing of connection.
      # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
      allow-insecure: false

    tlsCACerts:
      # Certificate location absolute path
      path: /home/harry/sidecar/plugins/fabric/crypto-config/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem

  peer1.org2.example.com:
    # this URL is used to send endorsement and query requests
    url: grpcs://peer1.org2.example.com:10051
    eventUrl: grpcs://peer1.org2.example.com:10053
    grpcOptions:
      ssl-target-name-override: peer1.org2.example.com
      # These parameters should be set in coordination with the keepalive policy on the server,
      # as incompatible settings can result in closing of connection.
      # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
      allow-insecure: false

    tlsCACerts:
      # Certificate location absolute path
      path: /home/harry/sidecar/plugins/fabric/crypto-config/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem

  #
  # CA is a special kind of Certificate Authority provided by Hyperledger Fab which allows
  # certificate management to be done via REST APIs. Application may choose to use a standard
  # Certificate Authority instead of CA, in which case this section would not be specified.
  #
  # certificateAuthorities:
  #   ca.org1.example.com:
  #     url: https://ca.org1.example.com:7053
  #     tlsCACerts:
  #       # Comma-Separated list of paths
  #       path: /home/shaojie/config_v1/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem
  #       # Client key and cert for SSL handshake with Fab CA
  #       client:
  #         key:
  #           path: /home/shaojie/config_v1/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/client.key
  #         cert:
  #           path: /home/shaojie/config_v1/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/tls/client.crt

  #     # CA supports dynamic user enrollment via REST APIs. A "root" user, a.k.a registrar, is
  #     # needed to enroll and invoke new users.
  #     registrar:
  #       enrollId: admin
  #       enrollSecret: adminpw
  #     # [Optional] The optional name of the CA.
  #     caName: ca.org1.example.com


  # EntityMatchers enable substitution of network hostnames with static configurations
  # so that properties can be mapped. Regex can be used for this purpose
  # UrlSubstitutionExp can be empty which means the same network hostname will be used
  # UrlSubstitutionExp can be given same as mapped peer url, so that mapped peer url can be used
  # UrlSubstitutionExp can have golang regex matchers like 1.local.example.2:3 for pattern
  # like peer0.fbi.citizens.com:1234 which converts peer0.fbi.citizens.com to peer0.FBI.local.citizens.com:1234
  # sslTargetOverrideUrlSubstitutionExp follow in the same lines as
  # SubstitutionExp for the fields gprcOptions.ssl-target-name-override respectively
# In any case mappedHost's config will be used, so mapped host cannot be empty, if entityMatchers are used
#entityMatchers:
entityMatchers:
  peer:
  - pattern: (\w*)peer0.org1.example.com:(\w*)
    urlSubstitutionExp: grpcs://peer0.org1.example.com:7051
    eventUrlSubstitutionExp: grpcs://peer0.org1.example.com:7053
    sslTargetOverrideUrlSubstitutionExp: peer0.org1.example.com
    mappedHost: peer0.org1.example.com


  - pattern: (\w*)peer1.org1.example.com:(\w*)
    urlSubstitutionExp: grpcs://peer1.org1.example.com:8051
    eventUrlSubstitutionExp: grpcs://peer1.org1.example.com:8053
    sslTargetOverrideUrlSubstitutionExp: peer1.org1.example.com
    mappedHost: peer1.org1.example.com

  - pattern: (\w*)peer0.org2.example.com:(\w*)
    urlSubstitutionExp: grpcs://peer0.org2.example.com:9051
    eventUrlSubstitutionExp: grpcs://peer0.org2.example.com:9053
    sslTargetOverrideUrlSubstitutionExp: peer0.org2.example.com
    mappedHost: peer0.org2.example.com

  - pattern: (\w*)peer1.org2.example.com:(\w*)
    urlSubstitutionExp: grpcs://pee1.org2.example.com:10051
    eventUrlSubstitutionExp: grpcs://pee1.org2.example.com:10053
    sslTargetOverrideUrlSubstitutionExp: peer1.org2.example.com
    mappedHost: peer1.org2.example.com

  #  orderer:
  #    - pattern: (\w+).example.(\w+)
  #      urlSubstitutionExp: orderer.citizens.com:7050
  #      sslTargetOverrideUrlSubstitutionExp: orderer.citizens.com
  #      mappedHost: orderer.citizens.com
  #
  #    - pattern: (\w+).example2.(\w+)
  #      urlSubstitutionExp: localhost:7050
  #      sslTargetOverrideUrlSubstitutionExp: localhost
  #      mappedHost: orderer.citizens.com
  #
  #    - pattern: (\w+).example3.(\w+)
  #      urlSubstitutionExp:
  #      sslTargetOverrideUrlSubstitutionExp:
  #      mappedHost: orderer.citizens.com
  #
  #    - pattern: (\w+).example4.(\w+):(\d+)
  #      urlSubstitutionExp: 1.example.2:3
  #      sslTargetOverrideUrlSubstitutionExp: 1.example.2
  #      mappedHost: orderer.citizens.com
  #
  #  certificateAuthority:
  #    - pattern: (\w+).fbi.citizens.com.(\w+)
  #      urlSubstitutionExp:
  #      mappedHost: ca.fbi.citizens.com
  #
  #entityMatchers:
  orderer:
  - pattern: (\w*)orderer.example.com(\w*)
    urlSubstitutionExp: grpcs://orderer.example.com:7050
    sslTargetOverrideUrlSubstitutionExp: orderer.example.com
    mappedHost: orderer.example.com

```

* 修改跨链合约配置fabric.toml

```
addr = "peer0.org2.example.com:9053" // 若Fabric部署在服务器上，该为服务器地址

event_filter = "interchain-event-name"

username = "Admin"

ccid = "broker" // 若部署跨链broker合约名字不是broker需要修改

channel_id = "mychannel"

org = "org2"
```


至此，fabric跨链插件部分配置完成
