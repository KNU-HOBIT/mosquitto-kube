# Mosquitto Kubernetes Deployment

## Description

Kubernetes configuration files for deploying Mosquitto, including configuration, passwords, deployment, and service.

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Files](#files)
- [Check Status](#check-status)

## Introduction

This repository contains the necessary Kubernetes configuration files to deploy an instance of the Mosquitto MQTT broker. The deployment includes a ConfigMap for the Mosquitto configuration, a ConfigMap for user passwords, a Deployment specification, a PersistentVolumeClaim, and a NodePort Service.

## Installation

To deploy Mosquitto on your Kubernetes cluster, follow these steps:

1. Ensure you have a running Kubernetes cluster and `kubectl` configured.
2. Apply the Kustomization:

    ```sh
    kubectl apply -k . -n mosquitto
    ```

## Usage

After deployment, Mosquitto will be available on port `30083` of your Kubernetes nodes. You can interact with the MQTT broker using an MQTT client.

## Configuration

### mosquitto-config.yaml

This file defines the Mosquitto configuration using a ConfigMap. Customize the `mosquitto.conf` section as needed.

### mosquitto-password.yaml

This file contains user passwords for Mosquitto in a ConfigMap. The `password.txt` file should follow the format `username:password`.


### mosquitto.yaml

This file defines the Mosquitto Service, PersistentVolumeClaim, and Deployment.

- **Service**: Exposes Mosquitto on port `1883` using NodePort `30083`.
- **PersistentVolumeClaim**: Requests 2Gi of storage using the `rook-cephfs` storage class.
- **Deployment**: Defines the Mosquitto pod configuration, including volume mounts for configuration and password files.



## Mosquitto용 해시된 패스워드 생성 방법

Mosquitto용 해시된 패스워드를 생성하려면 Mosquitto 브로커 패키지의 일부인 `mosquitto_passwd` 유틸리티를 사용할 수 있습니다. 다음 단계를 따라 해시된 패스워드를 생성하세요:

### `mosquitto_passwd` 유틸리티 사용

1. **Mosquitto 설치**: Mosquitto가 설치되어 있지 않다면 패키지 관리자를 사용하여 설치하세요.

   - **Ubuntu/Debian**:
     ```sh
     sudo apt-get install mosquitto mosquitto-clients
     ```

   - **CentOS/RHEL**:
     ```sh
     sudo yum install mosquitto mosquitto-clients
     ```

2. **패스워드 파일 생성**: `mosquitto_passwd` 유틸리티를 사용하여 패스워드 파일을 생성합니다.

    ```sh
    mosquitto_passwd -c password.txt admin
    ```
    ```
    Password: # 생성하고자하는 비밀번호 생성
    Reenter password: # 비밀번호 확인
    ```
    -> 현재 경로에 `password.txt`가 생성 완료.
    
3. **해시된 패스워드 보기**: `password.txt` 파일을 열어 해시된 패스워드를 확인합니다.

   ```sh
   cat password.txt
    ```
4. **해시된 패스워드 사용**: `password.txt` 파일의 내용을 복사하여 Kubernetes ConfigMap (`mosquitto-password.yaml`)에 붙여넣습니다.

    Ex. `mosquitto-password.yaml` 파일

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: mosquitto-password
      labels:
        app: mosquitto
    data:
      password.txt: |
        admin:$6$Dkc/7RZS6eAUAU9 ... Iw8Vqg==
    ```


## Files

- `kustomization.yaml`: Lists the resources to be deployed.
- `mosquitto-config.yaml`: ConfigMap for Mosquitto configuration.
- `mosquitto-password.yaml`: ConfigMap for Mosquitto passwords.
- `mosquitto.yaml`: Service, PersistentVolumeClaim, and Deployment for Mosquitto.

## Check Status

To verify that Mosquitto is running, you can use the following command to check the status of the pods:

```sh
kubectl get pods -l app=mosquitto -n mosquitto
```