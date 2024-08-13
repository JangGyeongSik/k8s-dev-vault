# Kubernetes에서 Vault HA 구성하기

## Vault 초기화 및 볼트인 해제

### Purpose
---
이 단계에서는 Vault 서버를 초기화하고 초기 키 및 루트 토큰을 얻을 것입니다.

### Requirement
---
- Vault 클러스터가 실행 중인 Kubernetes 클러스터
- Vault 클라이언트 (CLI)
- Kubernetes API에 액세스할 수 있는 kubeconfig 파일
- [Valut Helm Chart](./vault/)
- [Vault Ingress](./vault-ingress/)

### Vault 초기화
먼저 Vault 클러스터를 초기화해야 합니다. 이 작업은 Vault를 사용할 수 있도록 설정하고, 초기 키와 루트 토큰을 생성합니다.


1. Vault를 초기화합니다.
    ```bash
    vault operator init
    kubectl exec -it vault-0 -- vault operator init
    kubectl exec -it vault-0 -- vault operator unseal
    ```
    - 위 과정을 반복하여 unseal을 진행해줍니다.
    - [Vault Unseal에 대한 설명](https://developer.hashicorp.com/vault/docs/concepts/seal)

2. 초기화가 완료되면 출력되는 초기 키 및 루트 토큰을 안전한 곳에 보관합니다. 이들은 나중에 Vault에 대한 액세스를 얻거나 복구할 때 필요합니다.

3. Vault의 상태를 확인하여 초기화가 성공적으로 완료되었는지 확인합니다.
    ```bash
    vault status
    ```

4. Vault Join
    - 각 Pod에 접근하여 Raft Join을 진행해서, Leader가 사용이 불가능할때 Voter를 자동으로 Leader로 승격시켜주게끔 설정합니다. 
    ```bash
    vault operator raft join http://vault-0.vault-internal:8200
    ```
5. Vault에 대한 루트 토큰을 사용하여 로그인합니다.
    ```bash
    vault login <root-token>
    ```


Vault를 초기화하고 성공적으로 사용할 수 있는 상태로 만들었습니다.

이제 Vault에 대한 액세스를 얻었으므로 암호화 된 값을 안전하게 저장하고 애플리케이션에서 사용할 수 있습니다.
