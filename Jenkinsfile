podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:jdk8
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
      - name: centos
        image: centos
        command:
        - sleep
        args:
        - 99d
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
''') {
  node(POD_LABEL) {
    stage('k8s') {
       git 'https://github.com/vijayvad/chapter09/tree/feature/sample1'
      container('centos') {
        stage('start calculator') {
          sh '''
          curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
          chmod +x ./kubectl
          ./kubectl apply -f calculator.yaml
          ./kubectl apply -f hazelcast.yaml
          '''
        }
      }
    }
    stage('gradle') {
      git 'https://github.com/vijayvad/chapter09/tree/feature/sample3'
      container('gradle') {
        stage('test calculator') {
          sh '''
          chmod +x gradlew
          test $(curl calculator-service:8080/sum?a=1\\&b=2) -eq 3
          test $(curl calculator-service:8080/div?a=10\\&b=10) -eq 1
          ./gradlew acceptanceTest -Dcalculator.url=http://calculator-service:8080
          cat build/reports/tests/acceptanceTest/index.html
          '''
        }
      }
    }
  }
}
