
```yaml
- name: day14_epm_assignment-ui-deployer
  type: com.sap.application.content
  path: .
  requires:
    - name: day14_epm_assignment-html5-repo-host
      parameters:
        content-target: true
  build-parameters:
    build-result: resources
    requires:
      - name: day14_epm_assignment-app
        artifacts:
          - day14_epm_assignment-app.zip
        target-path: resources/