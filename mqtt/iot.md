
# === Separate subscriber, publisher, and broker containers

# remove any existing container
docker container stop mqtt_broker ; docker container rm mqtt_broker
docker container stop node_red ; docker container rm node_red


# Create a container running in the background
docker container run -d --name mqtt_broker -p 1883:1883 mosquitto sleep inf


# Start a broker in one terminal
docker container exec mqtt_broker mosquitto -c /etc/mosquitto/mosquitto.conf -v -d


# Create a subscriber in a second terminal
mqtt_ip=$( docker container inspect mqtt_broker | jq -r .[0].NetworkSettings.Gateway )
docker container run --rm --name mqtt_sub mosquitto \
  mosquitto_sub \
    --host ${mqtt_ip} \
    --port 1883 \
    --topic CNMI/IoT


# Create a publisher in a third terminal
mqtt_ip=$( docker container inspect mqtt_broker | jq -r .[0].NetworkSettings.Gateway )
docker container run --rm --name mqtt_pub -i mosquitto \
    mosquitto_pub \
      --host ${mqtt_ip} \
      --port 1883 \
      --topic CNMI/IoT \
      --message "Hello World"


# start node-red
docker container run -d --name node-red -p 1880:1880 mqtt-red node-red

mqtt_ip=$( docker container inspect mqtt_broker | jq -r .[0].NetworkSettings.Gateway )
echo ${mqtt_ip}
echo http://localhost:1880


