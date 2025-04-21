FROM frangoteam/fuxa

RUN apt update && apt install -y curl iproute2 iptables

COPY fuxa-project.json add-user-viewer.json add-user-operator.json settings.json /config/

EXPOSE 1881

CMD bash -c "\
  npm start & \
  pid=\$!; \
  echo '[*] Waiting for FUXA to become available on port 1881...'; \
  until curl -sf http://localhost:1881; do sleep 1; done; \
  echo '[+] FUXA is up. Seeding config...'; \
  curl -sf -X POST -H 'Content-Type: application/json' -d @/config/fuxa-project.json http://localhost:1881/api/project || echo '[!] Project upload failed'; \
  curl -sf -X POST -H 'Content-Type: application/json' -d @/config/add-user-viewer.json http://localhost:1881/api/users || echo '[!] Viewer upload failed'; \
  curl -sf -X POST -H 'Content-Type: application/json' -d @/config/add-user-operator.json http://localhost:1881/api/users || echo '[!] Operator upload failed'; \
  curl -sf -X POST -H 'Content-Type: application/json' -d @/config/settings.json http://localhost:1881/api/settings || echo '[!] Settings upload failed'; \
  echo '[+] Config complete. Bringing FUXA to foreground.'; \
  wait \$pid"