#2
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world-job
spec:
  parallelism: 1    
  completions: 1    
  template:         
    metadata:
      name: hello-world-job
    spec:
      volumes:
      - name: hello-world-scripts-volume
        configMap:
          name: hello-world-scripts
      containers:
      - name: hello-world-job
        image: alpine
        volumeMounts:
          - mountPath: /hello-world-scripts
            name: hello-world-scripts-volume
        env:
          - name: HOME
            value: /tmp
        command:
        - /bin/sh
        - -c
        - |
          echo "scripts in /hello-world-scripts"
          ls -lh /hello-world-scripts
          echo "copy scripts to /tmp"
          cp /hello-world-scripts/*.sh /tmp
          echo "apply 'chmod +x' to /tmp/*.sh"
          chmod +x /tmp/*.sh
          echo "execute script-one.sh now"
          /tmp/script-one.sh
      restartPolicy: Never
---
apiVersion: v1
items:
- apiVersion: v1
  data:
    script-one.sh: |
      echo "script-one.sh"
      date
      sleep 1
      echo "run /tmp/script-2.sh now"
      /tmp/script-2.sh
    script-2.sh: |
      echo "script-2.sh"
      sleep 1
      date
  kind: ConfigMap
  metadata:
    creationTimestamp: null
    name:  hello-world-scripts
kind: List
metadata: {}
