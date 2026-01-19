# n8n-demos
Creating multi agent workflows in n8n

## Running n8n in container
`
docker pull docker.n8n.io/n8nio/n8n:latest  
`

`
docker volume create n8n_demos_data
`
```

docker run -it --rm \
 --name n8n-demos \
 -p 5678:5678 \
 -e GENERIC_TIMEZONE="CET" \
 -e TZ="CET" \
 -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
 -e N8N_RUNNERS_ENABLED=true \
 -v n8n_demos_data:/home/node/.n8n \
 docker.n8n.io/n8nio/n8n:latest
```


### n8n mcp server
`
docker pull ghcr.io/czlonkowski/n8n-mcp:latest
`