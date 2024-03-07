# Pour compiler depuis du code présent dans mon répertoire local MAIS en utilisant une toolchain go d'une image Docker
# docker container run --volume /workspace/20240307_test/color:/app golang go build -o /app/color /app/color.go

# Pour exécuter l binaire depuis une image custom forgée ci-dessous
# docker build --tag myImage .
# docker run -P myImage

FROM busybox

WORKDIR /
COPY ./color /

EXPOSE 80
CMD /color

# Pour requêter depuis ma machine locale
# curl http://localhost:32xxx/

# où 32xxx est le port sur lequel Docker binde dynamiquement le port 80 exposé par le container.