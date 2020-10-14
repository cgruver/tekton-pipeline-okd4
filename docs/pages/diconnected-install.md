# WIP: Not ready to use

```bash
mkdir -p ~/tkn-workdir
cp -r ./disconnected-install/addons ~/tkn-workdir
cp ./disconnected-install/*.yaml ~/tkn-workdir

LOCAL_REGISTRY=nexus.your.domain.org:5000
for i in $(cat tkn-images)
do 
    IMAGE_TAG=${LOCAL_REGISTRY}/$(echo $i | cut -d"/" -f2-)
    docker pull ${i}
    docker tag ${i} ${IMAGE_TAG}
done

docker login ${LOCAL_REGISTRY}

for i in $(cat tkn-images)
do 
    IMAGE_TAG=${LOCAL_REGISTRY}/$(echo $i | cut -d"/" -f2-)
    docker push ${IMAGE_TAG}
done

cd ~/tkn-workdir
for i in $(find . | grep yaml)
do
    sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" ${i}
done

```