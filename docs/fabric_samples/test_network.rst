**********************
Fabric Test Network
**********************

如何使用测试网络（test network）请阅读官方文档 `Using the Fabric test network <https://hyperledger-fabric.readthedocs.io/en/release-2.5/test_network.html>`_ 。

测试网络介绍
=============

下载 Hyperledger Fabric Docker images 和 samples 后，可以使用 fabric-samples 库中提供的脚本部署测试网络（test network）。测试网络是为了通过在本地计算机上运行节点来了解 Fabric。开发人员可以使用网络来测试智能合约和应用程序。该网络仅用作教育和测试工具，而不是用作如何建立网络的模型。一般来说，不鼓励对脚本进行修改，否则可能会破坏网络。**它基于有限的配置，不应用作部署生产网络的模板。**

在启动测试网络前，按照 `Getting Started - Install <https://hyperledger-fabric.readthedocs.io/en/release-2.5/getting_started.html>`_ 安装所需的软件。

test-network 目录
===================

启动测试网络所需的文件在 fabric-samples 库下的 test-network 目录中，使用命令 ``cd fabric-samples/test-network`` 进入此目录。

network.sh 文件
-----------------

`文件链接 <https://github.com/hyperledger/fabric-samples/blob/v2.4.9/test-network/network.sh>`_

在此目录中，可以找到脚本 network.sh，它使用本地计算机上的 Docker Image 建立 Fabric 网络。运行 ``./network.sh -h`` 来打印脚本帮助文档：

.. code-block:: text

    Usage:
    network.sh <Mode> [Flags]
        Modes:
            up - Bring up Fabric orderer and peer nodes. No channel is created
            up createChannel - Bring up fabric network with one channel
            createChannel - Create and join a channel after the network is created
            deployCC - Deploy a chaincode to a channel (defaults to asset-transfer-basic)
            down - Bring down the network

        Flags:
            Used with network.sh up, network.sh createChannel:
            -ca <use CAs> -  Use Certificate Authorities to generate network crypto material
            -c <channel name> - Name of channel to create (defaults to "mychannel")
            -s <dbtype> - Peer state database to deploy: goleveldb (default) or couchdb
            -r <max retry> - CLI times out after certain number of attempts (defaults to 5)
            -d <delay> - CLI delays for a certain number of seconds (defaults to 3)
            -i <imagetag> - Docker image tag of Fabric to deploy (defaults to "latest")
            -cai <ca_imagetag> - Docker image tag of Fabric CA to deploy (defaults to "latest")
            -verbose - Verbose mode

            Used with network.sh deployCC
            -c <channel name> - Name of channel to deploy chaincode to
            -ccn <name> - Chaincode name.
            -ccl <language> - Programming language of the chaincode to deploy: go (default), java, javascript, typescript
            -ccv <version>  - Chaincode version. 1.0 (default), v2, version3.x, etc
            -ccs <sequence>  - Chaincode definition sequence. Must be an integer, 1 (default), 2, 3, etc
            -ccp <path>  - File path to the chaincode.
            -ccep <policy>  - (Optional) Chaincode endorsement policy using signature policy syntax. The default policy requires an endorsement from Org1 and Org2
            -cccg <collection-config>  - (Optional) File path to private data collections configuration file
            -cci <fcn name>  - (Optional) Name of chaincode initialization function. When a function is provided, the execution of init will be requested and the function will be invoked.

            -h - Print this message

    Possible Mode and flag combinations
        up -ca -r -d -s -i -cai -verbose
        up createChannel -ca -c -r -d -s -i -cai -verbose
        createChannel -c -r -d -verbose
        deployCC -ccn -ccl -ccv -ccs -ccp -cci -r -d -verbose

    Examples:
        network.sh up createChannel -ca -c mychannel -s couchdb -i 2.0.0
        network.sh createChannel -c channelName
        network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
        network.sh deployCC -ccn mychaincode -ccp ./user/mychaincode -ccv 1 -ccl javascript

network.sh 脚本从362行开始定义变量和执行函数，在此之前的代码基本上是定义了一些函数。

.. code-block:: bash

    # 初始化脚本所需参数的默认值

    # Using crpto vs CA. default is cryptogen
    CRYPTO="cryptogen"
    # timeout duration - the duration the CLI should wait for a response from
    # another container before giving up
    MAX_RETRY=5
    # default for delay between commands
    CLI_DELAY=3
    # channel name defaults to "mychannel"
    CHANNEL_NAME="mychannel"
    # chaincode name defaults to "NA"
    CC_NAME="NA"
    # chaincode path defaults to "NA"
    CC_SRC_PATH="NA"
    # endorsement policy defaults to "NA". This would allow chaincodes to use the majority default policy.
    CC_END_POLICY="NA"
    # collection configuration defaults to "NA"
    CC_COLL_CONFIG="NA"
    # chaincode init function defaults to "NA"
    CC_INIT_FCN="NA"
    # use this as the default docker-compose yaml definition
    COMPOSE_FILE_BASE=compose-test-net.yaml
    # docker-compose.yaml file if you are using couchdb
    COMPOSE_FILE_COUCH=compose-couch.yaml
    # certificate authorities compose file
    COMPOSE_FILE_CA=compose-ca.yaml
    # use this as the default docker-compose yaml definition for org3
    COMPOSE_FILE_ORG3_BASE=compose-org3.yaml
    # use this as the docker compose couch file for org3
    COMPOSE_FILE_ORG3_COUCH=compose-couch-org3.yaml
    # certificate authorities compose file
    COMPOSE_FILE_ORG3_CA=compose-ca-org3.yaml
    # chaincode language defaults to "NA"
    CC_SRC_LANGUAGE="NA"
    # default to running the docker commands for the CCAAS
    CCAAS_DOCKER_RUN=true
    # Chaincode version
    CC_VERSION="1.0"
    # Chaincode definition sequence
    CC_SEQUENCE=1
    # default database
    DATABASE="leveldb"

    # Get docker sock path from environment variable
    SOCK="${DOCKER_HOST:-/var/run/docker.sock}"
    DOCKER_SOCK="${SOCK##unix://}"

    # Parse commandline args

    # 如果 ./network.sh 不指定参数，就打印帮助文档，否则把第一个参数赋值为 MODE 变量，并舍去第一个参数。
    ## Parse mode
    if [[ $# -lt 1 ]] ; then
        printHelp
        exit 0
    else
        MODE=$1
        shift
    fi

    # ./network up createChannel 情况，up 赋值为了上述的 MODE，如果剩下的第一个参数是 createChannel，
    # 修改 MODE 为 createChannel，舍去第一个参数。
    # parse a createChannel subcommand if used
    if [[ $# -ge 1 ]] ; then
        key="$1"
        if [[ "$key" == "createChannel" ]]; then
            export MODE="createChannel"
            shift
        fi
    fi

    # 解析剩下的 flags，即替换掉参数的默认值。
    # parse flags
    while [[ $# -ge 1 ]] ; do
        key="$1"
        case $key in
        -h )
            printHelp $MODE
            exit 0
            ;;
        -c )
            CHANNEL_NAME="$2"
            shift
            ;;
        -ca )
            CRYPTO="Certificate Authorities"
            ;;
        -r )
            MAX_RETRY="$2"
            shift
            ;;
        -d )
            CLI_DELAY="$2"
            shift
            ;;
        -s )
            DATABASE="$2"
            shift
            ;;
        -ccl )
            CC_SRC_LANGUAGE="$2"
            shift
            ;;
        -ccn )
            CC_NAME="$2"
            shift
            ;;
        -ccv )
            CC_VERSION="$2"
            shift
            ;;
        -ccs )
            CC_SEQUENCE="$2"
            shift
            ;;
        -ccp )
            CC_SRC_PATH="$2"
            shift
            ;;
        -ccep )
            CC_END_POLICY="$2"
            shift
            ;;
        -cccg )
            CC_COLL_CONFIG="$2"
            shift
            ;;
        -cci )
            CC_INIT_FCN="$2"
            shift
            ;;
        -ccaasdocker )
            CCAAS_DOCKER_RUN="$2"
            shift
            ;;
        -verbose )
            VERBOSE=true
            ;;
        * )
            errorln "Unknown flag: $key"
            printHelp
            exit 1
            ;;
        esac
        shift
    done

    # Are we generating crypto material with this command?
    if [ ! -d "organizations/peerOrganizations" ]; then
        CRYPTO_MODE="with crypto from '${CRYPTO}'"
    else
        CRYPTO_MODE=""
    fi

    # Determine mode of operation and printing out what we asked for
    if [ "$MODE" == "up" ]; then
        # ./network.sh up 会进入此分支，执行 networkUp 函数。
        infoln "Starting nodes with CLI timeout of '${MAX_RETRY}' tries and CLI delay of '${CLI_DELAY}' seconds and using database '${DATABASE}' ${CRYPTO_MODE}"
        networkUp
    elif [ "$MODE" == "createChannel" ]; then
        # ./network.sh [createChannel | up createChannel] 会进入此分支。
        infoln "Creating channel '${CHANNEL_NAME}'."
        infoln "If network is not up, starting nodes with CLI timeout of '${MAX_RETRY}' tries and CLI delay of '${CLI_DELAY}' seconds and using database '${DATABASE} ${CRYPTO_MODE}"
        createChannel
    elif [ "$MODE" == "down" ]; then
        infoln "Stopping network"
        networkDown
    elif [ "$MODE" == "restart" ]; then
        infoln "Restarting network"
        networkDown
        networkUp
    elif [ "$MODE" == "deployCC" ]; then
        infoln "deploying chaincode on channel '${CHANNEL_NAME}'"
        deployCC
    elif [ "$MODE" == "deployCCAAS" ]; then
        infoln "deploying chaincode-as-a-service on channel '${CHANNEL_NAME}'"
        deployCCAAS
    else
        printHelp
        exit 1
    fi

createChannel 函数：

.. code-block:: bash

    # call the script to create the channel, join the peers of org1 and org2,
    # and then update the anchor peers for each organization
    function createChannel() {
        # Bring up the network if it is not already up.
        bringUpNetwork="false"

        # 判断 docker 命令是否存在
        if ! $CONTAINER_CLI info > /dev/null 2>&1 ; then
            fatalln "$CONTAINER_CLI network is required to be running to create a channel"
        fi

        # 获取运行中的 hyperledger 容器数量。
        # check if all containers are present
        CONTAINERS=($($CONTAINER_CLI ps | grep hyperledger/ | awk '{print $2}'))
        len=$(echo ${#CONTAINERS[@]})

        # 如果容器数量大于等于4，并且 organizations/peerOrganizations 目录不存在，执行 networkDown 函数。
        if [[ $len -ge 4 ]] && [[ ! -d "organizations/peerOrganizations" ]]; then
            echo "Bringing network down to sync certs with containers"
            networkDown
        fi

        # 如果容器数量小于4或者目录不存在，bringUpNetwork 赋值为 true。
        [[ $len -lt 4 ]] || [[ ! -d "organizations/peerOrganizations" ]] && bringUpNetwork="true" || echo "Network Running Already"

        # bringUpNetwork 如果为 true，调用 networkUp函数。
        if [ $bringUpNetwork == "true"  ]; then
            infoln "Bringing up network"
            networkUp
        fi

        # 调用 scripts/createChannel.sh 脚本。
        # now run the script that creates a channel. This script uses configtxgen once
        # to create the channel creation transaction and the anchor peer updates.
        scripts/createChannel.sh $CHANNEL_NAME $CLI_DELAY $MAX_RETRY $VERBOSE
    }

networkDown 函数

.. code-block:: bash

    # Tear down running network
    function networkDown() {
        # docker-compose.yaml 配置文件
        COMPOSE_BASE_FILES="-f compose/${COMPOSE_FILE_BASE} -f compose/${CONTAINER_CLI}/${CONTAINER_CLI}-${COMPOSE_FILE_BASE}"
        COMPOSE_COUCH_FILES="-f compose/${COMPOSE_FILE_COUCH} -f compose/${CONTAINER_CLI}/${CONTAINER_CLI}-${COMPOSE_FILE_COUCH}"
        COMPOSE_CA_FILES="-f compose/${COMPOSE_FILE_CA} -f compose/${CONTAINER_CLI}/${CONTAINER_CLI}-${COMPOSE_FILE_CA}"
        COMPOSE_FILES="${COMPOSE_BASE_FILES} ${COMPOSE_COUCH_FILES} ${COMPOSE_CA_FILES}"

        # stop org3 containers also in addition to org1 and org2, in case we were running sample to add org3
        COMPOSE_ORG3_BASE_FILES="-f addOrg3/compose/${COMPOSE_FILE_ORG3_BASE} -f addOrg3/compose/${CONTAINER_CLI}/${CONTAINER_CLI}-${COMPOSE_FILE_ORG3_BASE}"
        COMPOSE_ORG3_COUCH_FILES="-f addOrg3/compose/${COMPOSE_FILE_ORG3_COUCH} -f addOrg3/compose/${CONTAINER_CLI}/${CONTAINER_CLI}-${COMPOSE_FILE_ORG3_COUCH}"
        COMPOSE_ORG3_CA_FILES="-f addOrg3/compose/${COMPOSE_FILE_ORG3_CA} -f addOrg3/compose/${CONTAINER_CLI}/${CONTAINER_CLI}-${COMPOSE_FILE_ORG3_CA}"
        COMPOSE_ORG3_FILES="${COMPOSE_ORG3_BASE_FILES} ${COMPOSE_ORG3_COUCH_FILES} ${COMPOSE_ORG3_CA_FILES}"

        if [ "${CONTAINER_CLI}" == "docker" ]; then
            # 删除 docker-compose.yaml 文件中的容器和 volume，配置文件中用到了 DOCKER_SOCK 环境变量。
            DOCKER_SOCK=$DOCKER_SOCK ${CONTAINER_CLI_COMPOSE} ${COMPOSE_FILES} ${COMPOSE_ORG3_FILES} down --volumes --remove-orphans
        elif [ "${CONTAINER_CLI}" == "podman" ]; then
            ${CONTAINER_CLI_COMPOSE} ${COMPOSE_FILES} ${COMPOSE_ORG3_FILES} down --volumes
        else
            fatalln "Container CLI  ${CONTAINER_CLI} not supported"
        fi


        # Don't remove the generated artifacts -- note, the ledgers are always removed
        if [ "$MODE" != "restart" ]; then
            # Bring down the network, deleting the volumes
            ${CONTAINER_CLI} volume rm docker_orderer.example.com docker_peer0.org1.example.com docker_peer0.org2.example.com
            #Cleanup the chaincode containers
            clearContainers
            #Cleanup images
            removeUnwantedImages
            #
            ${CONTAINER_CLI} kill $(${CONTAINER_CLI} ps -q --filter name=ccaas) || true
            # remove orderer block and other channel configuration transactions and certs
            ${CONTAINER_CLI} run --rm -v "$(pwd):/data" busybox sh -c 'cd /data && rm -rf system-genesis-block/*.block organizations/peerOrganizations organizations/ordererOrganizations'
            ## remove fabric ca artifacts
            ${CONTAINER_CLI} run --rm -v "$(pwd):/data" busybox sh -c 'cd /data && rm -rf organizations/fabric-ca/org1/msp organizations/fabric-ca/org1/tls-cert.pem organizations/fabric-ca/org1/ca-cert.pem organizations/fabric-ca/org1/IssuerPublicKey organizations/fabric-ca/org1/IssuerRevocationPublicKey organizations/fabric-ca/org1/fabric-ca-server.db'
            ${CONTAINER_CLI} run --rm -v "$(pwd):/data" busybox sh -c 'cd /data && rm -rf organizations/fabric-ca/org2/msp organizations/fabric-ca/org2/tls-cert.pem organizations/fabric-ca/org2/ca-cert.pem organizations/fabric-ca/org2/IssuerPublicKey organizations/fabric-ca/org2/IssuerRevocationPublicKey organizations/fabric-ca/org2/fabric-ca-server.db'
            ${CONTAINER_CLI} run --rm -v "$(pwd):/data" busybox sh -c 'cd /data && rm -rf organizations/fabric-ca/ordererOrg/msp organizations/fabric-ca/ordererOrg/tls-cert.pem organizations/fabric-ca/ordererOrg/ca-cert.pem organizations/fabric-ca/ordererOrg/IssuerPublicKey organizations/fabric-ca/ordererOrg/IssuerRevocationPublicKey organizations/fabric-ca/ordererOrg/fabric-ca-server.db'
            ${CONTAINER_CLI} run --rm -v "$(pwd):/data" busybox sh -c 'cd /data && rm -rf addOrg3/fabric-ca/org3/msp addOrg3/fabric-ca/org3/tls-cert.pem addOrg3/fabric-ca/org3/ca-cert.pem addOrg3/fabric-ca/org3/IssuerPublicKey addOrg3/fabric-ca/org3/IssuerRevocationPublicKey addOrg3/fabric-ca/org3/fabric-ca-server.db'
            # remove channel and script artifacts
            ${CONTAINER_CLI} run --rm -v "$(pwd):/data" busybox sh -c 'cd /data && rm -rf channel-artifacts log.txt *.tar.gz'
        fi
    }

networkUp 函数

.. code-block:: bash

    # Bring up the peer and orderer nodes using docker compose.
    function networkUp() {
        # 检查 peer 命令和其所需的配置文件是否存在；
        # 检查 peer 命令的版本和 docker image 版本是否一致；
        # 如果需要用到 Fabric CA，则检查 fabric-ca-client 命令是否存在；
        # 检查 fabric-ca-client 命令版本和 docker image 版本是否一致。
        checkPrereqs

        # generate artifacts if they don't exist
        if [ ! -d "organizations/peerOrganizations" ]; then
            # 默认使用 cryptogen 工具创建，也可以使用 Fabric CA 工具。
            # 创建 organization、peer、orderer、admin、user 对应的 msp 文件。
            createOrgs
        fi

        # 包含 peer、orderer 等配置 docker-compose.yaml 文件
        COMPOSE_FILES="-f compose/${COMPOSE_FILE_BASE} -f compose/${CONTAINER_CLI}/${CONTAINER_CLI}-${COMPOSE_FILE_BASE}"

        if [ "${DATABASE}" == "couchdb" ]; then
            # 如果指定了数据库为 couchdb，则加入相应的 docker compose 配置文件。
            COMPOSE_FILES="${COMPOSE_FILES} -f compose/${COMPOSE_FILE_COUCH} -f compose/${CONTAINER_CLI}/${CONTAINER_CLI}-${COMPOSE_FILE_COUCH}"
        fi

        # 启动容器、volume、网络
        DOCKER_SOCK="${DOCKER_SOCK}" ${CONTAINER_CLI_COMPOSE} ${COMPOSE_FILES} up -d 2>&1

        $CONTAINER_CLI ps -a
        if [ $? -ne 0 ]; then
            fatalln "Unable to start network"
        fi
    }

scripts/createChannel.sh 文件
--------------------------------

.. code-block:: bash

    # imports
    # 导入一些环境变量。
    . scripts/envVar.sh
    # 导入 println、infoln、errorln 等打印函数。
    . scripts/utils.sh

    # 后续需要用到的变量，可以通过调用此文件传入，也可使用默认值。
    CHANNEL_NAME="$1"
    DELAY="$2"
    MAX_RETRY="$3"
    VERBOSE="$4"
    : ${CHANNEL_NAME:="mychannel"}
    : ${DELAY:="3"}
    : ${MAX_RETRY:="5"}
    : ${VERBOSE:="false"}

    : ${CONTAINER_CLI:="docker"}
    : ${CONTAINER_CLI_COMPOSE:="${CONTAINER_CLI}-compose"}
    infoln "Using ${CONTAINER_CLI} and ${CONTAINER_CLI_COMPOSE}"

    if [ ! -d "channel-artifacts" ]; then
        mkdir channel-artifacts
    fi

.. code-block:: bash

    FABRIC_CFG_PATH=${PWD}/configtx

    ## Create channel genesis block
    infoln "Generating channel genesis block '${CHANNEL_NAME}.block'"
    # 使用 configtxgen 命令创建 genesis block。
    createChannelGenesisBlock

    FABRIC_CFG_PATH=$PWD/../config/
    BLOCKFILE="./channel-artifacts/${CHANNEL_NAME}.block"

    ## Create channel
    infoln "Creating channel ${CHANNEL_NAME}"
    # 创建 channel 并将 orderer 节点加入其中。
    createChannel
    successln "Channel '$CHANNEL_NAME' created"

    ## Join all the peers to the channel
    # 加入两个 peer 节点。
    infoln "Joining org1 peer to the channel..."
    joinChannel 1
    infoln "Joining org2 peer to the channel..."
    joinChannel 2

    ## Set the anchor peers for each org in the channel
    # 将两个 peer 节点更新为 anchor peer，anchor peer 可以快速响应后续其他节点的配置更新，
    # 暂时不清楚为什么 peer 节点默认不是 anchor peer，需要手动执行命令。
    infoln "Setting anchor peer for org1..."
    setAnchorPeer 1
    infoln "Setting anchor peer for org2..."
    setAnchorPeer 2

    successln "Channel '$CHANNEL_NAME' joined"
