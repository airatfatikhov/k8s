## Change ConfigMap

# If change configmap
- [x] - add annotation to deployment <br>
``checksum/config: "{{ include (print $.Template.BasePath \"/configmap.yaml\") . | sha256sum }}"``