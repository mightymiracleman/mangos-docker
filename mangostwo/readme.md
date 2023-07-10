Creating a user:
when creating a user in the database:
the password is hashed as follows:
 select SHA1(CONCAT(UPPER('username'),':',UPPER('password')))


 K8 config;
 running into a lot of config issues with networking we need to expose ports 8085 and 3724

 Traefik is throwing fits: so far we've added IngressRouteTCP / IngressRouteUDP / IngressRoute for each port to the manifest kubernetes-deployment (no luck)

 I've also manually edited the deployment for traefik adding the following args to the Traefik deployment:
         - --entryPoints.tcp-3724.address=:3724/tcp
        - --entryPoints.udp-3724.address=:3724/udp
        - --entryPoints.tcp-8085.address=:8085/tcp
        - --entryPoints.udp-8085.address=:8085/udp

        This hasn't worked yet either

Finally I've added multiple containers to Klipper  svclb-trefik daemonset (traefik uses this to map cluster ports to host ports):

      - env:
        - name: SRC_PORT
          value: "3724"
        - name: DEST_PROTO
          value: TCP
        - name: DEST_PORT
          value: "3724"
        - name: DEST_IP
          value: 10.43.182.168
        image: rancher/klipper-lb:v0.2.0
        imagePullPolicy: IfNotPresent
        name: lb-port-3724
        ports:
        - containerPort: 3724
          hostPort: 3724
          name: lb-port-3724
          protocol: TCP
        resources: {}
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File

      - env:
        - name: SRC_PORT
          value: "8085"
        - name: DEST_PROTO
          value: TCP
        - name: DEST_PORT
          value: "8085"
        - name: DEST_IP
          value: 10.43.182.168
        image: rancher/klipper-lb:v0.2.0
        imagePullPolicy: IfNotPresent
        name: lb-port-8085
        ports:
        - containerPort: 8085
          hostPort: 8085
          name: lb-port-8085
          protocol: TCP
        resources: {}
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File



