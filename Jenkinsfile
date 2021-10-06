pipeline {
  agent {
    node {
      label 'maven'
    }

  }
  stages {
    stage('Release') {
      steps {
        sh '''ls 
oc project react-intro2
oc start-build simple-java-maven-app  --follow --wait -e OUTPUT_IMAGE=${BUILD_NUMBER}'''
      }
    }

    stage('Docker_lint') {
      agent {
        node {
          label 'nodejs'
        }

      }
      steps {
        sh '''npm install -g dockerlint
dockerlint Dockerfile'''
      }
    }

    stage('Set parameter') {
      agent {
        node {
          label 'maven'
        }

      }
      steps {
        sh 'properties([[$class: \'JiraProjectProperty\'], [$class: \'BuildConfigProjectProperty\', name: \'\', namespace: \'\', resourceVersion: \'\', uid: \'\'], parameters([string(\'git-url\'), string(defaultValue: \' master\', name: \' git-revision\'), string(defaultValue: \'/source\', name: \'source-dir\'), string(defaultValue: \'""\', name: \'image-url\'), string(defaultValue: \'""\', name: \'app-name\'), string(defaultValue: \'"route"\', name: \'deploy-ingress-type\'), string(description: \'The url of the helm repository\', name: \'helm-curl\')])])'
        sh '''set +x
        if [[ -n "${GIT_USERNAME}" ]] && [[ -n "${GIT_PASSWORD}" ]]; then
            git clone "$(echo $(params.git-url) | awk -F \'://\' \'{print $1}\')://${GIT_USERNAME}:${GIT_PASSWORD}@$(echo $(params.git-url) | awk -F \'://\' \'{print $2}\')" $(params.source-dir)
        else
            set -x
            git clone $(params.git-url) $(params.source-dir)
        fi
        set -x
        cd $(params.source-dir)
        git checkout $(params.git-revision)'''
        sh '''#!/usr/bin/env bash
          set -ex
          CURL_FLAGS="$(params.helm-curl)"
          echo "1. Package Helm Chart"
          GIT_URL="$(params.git-url)"
          GIT_REVISION="$(params.git-revision)"
          IMAGE_SERVER="$(echo "$(params.image-url)" | awk -F / \'{print $1}\')"
          IMAGE_NAMESPACE="$(echo "$(params.image-url)" | awk -F / \'{print $2}\')"
          IMAGE_REPOSITORY="$(echo "$(params.image-url)" | awk -F / \'{print $3}\' | awk -F : \'{print $1}\')"
          IMAGE_VERSION="$(echo "$(params.image-url)" | awk -F / \'{print $3}\' | awk -F : \'{print $2}\')"
          IMAGE_URL="${IMAGE_SERVER}/${IMAGE_NAMESPACE}/${IMAGE_REPOSITORY}"
          APP_NAME="$(params.app-name)"
          if [[ -z "${APP_NAME}" ]]; then
            APP_NAME="${IMAGE_REPOSITORY}"
          fi
          INGRESS_TYPE="$(params.deploy-ingress-type)"
          if [[ "${INGRESS_TYPE}" == "route" ]]; then
            ROUTE_ENABLED="true"
            INGRESS_ENABLED="false"
          else
            ROUTE_ENABLED="false"
            INGRESS_ENABLED="true"
          fi
          CHART_ROOT=$(find . -name chart)
          echo "CHART_ROOT: $CHART_ROOT"
          # Make a copy to be able to edit the chart
          mkdir -p "$(params.source-dir)/chart-temp"
          cp -R "$CHART_ROOT" "$(params.source-dir)/chart-temp/"
          CHART_ROOT="$(params.source-dir)/chart-temp/chart"
          CHART=$(find $CHART_ROOT -name \'Chart*.yaml\')
          echo "CHART: $CHART"
          CHART_NAME=$(cat $CHART | yq r - name)
          echo "CHART_NAME: $CHART_NAME"
          # Update Chart name and version
          if [[ "${CHART_NAME}" != "${APP_NAME}" ]]; then
            echo "Renaming chart from ${CHART_NAME} to ${APP_NAME}"
            cp -R "${CHART_ROOT}/${CHART_NAME}" "${CHART_ROOT}/${APP_NAME}"
            cat "${CHART_ROOT}/${CHART_NAME}/Chart.yaml" | \\
                yq w - name "${APP_NAME}" | \\
                yq w - version "${IMAGE_VERSION}" > "${CHART_ROOT}/${APP_NAME}/Chart.yaml"
          else
            echo "Chart name and image name match: ${APP_NAME}"
          fi
          CHART_NAME="${APP_NAME}"
          CHART_PATH="${CHART_ROOT}/${APP_NAME}"
          echo ""
          echo "Chart ${CHART_PATH}"
          cat ${CHART_PATH}/Chart.yaml
          RELEASE_NAME="${APP_NAME}"
          echo "RELEASE_NAME: $RELEASE_NAME"
          echo $(helm version)
          PREFIX=""
          if [[ -f "${CHART_PATH}/requirements.yaml" ]] || grep -Eq "^dependencies:" "${CHART_PATH}/Chart.yaml"; then
              DEPENDENCY_FILE="${CHART_PATH}/Chart.yaml"
              if [[ -f "${CHART_PATH}/requirements.yaml" ]]; then
                  DEPENDENCY_FILE="${CHART_PATH}/requirements.yaml"
              fi
              PREFIX="$(yq r -j "${DEPENDENCY_FILE}" | jq -r \'.dependencies | .[] | .alias // .name\' | head -1)."
          fi
          # Update helm chart with repository and tag values
          cat ${CHART_PATH}/values.yaml | \\
              yq w - "${PREFIX}nameOverride" "${APP_NAME}" | \\
              yq w - "${PREFIX}fullnameOverride" "${APP_NAME}" | \\
              yq w - "${PREFIX}vcsInfo.repoUrl" "${GIT_URL}" | \\
              yq w - "${PREFIX}vcsInfo.branch" "${GIT_REVISION}" | \\
              yq w - "${PREFIX}image.repository" "${IMAGE_URL}" | \\
              yq w - "${PREFIX}image.tag" "${IMAGE_VERSION}" | \\
              yq w - "${PREFIX}ingress.enabled" "${INGRESS_ENABLED}" | \\
              yq w - "${PREFIX}route.enabled" "${ROUTE_ENABLED}" > ./values.yaml.tmp
          cp ./values.yaml.tmp ${CHART_PATH}/values.yaml
          cat ${CHART_PATH}/values.yaml
          echo "CHECKING CHART (lint)"
          helm lint ${CHART_PATH}
          echo "2. Publish Helm Chart"
          if [[ -z "${HELM_URL}" ]] && [[ -z "${HELM_USER}" ]]; then
            if [[ -z "${ARTIFACTORY_URL}" ]]; then
              echo "It looks like Artifactory has not been installed (ARTIFACTORY_URL from artifactory-acess secret is missing). Skipping step."
              exit 0
            fi
            set +x
            if [[ -z "${ARTIFACTORY_USER}" ]]; then
              echo "Something\'s wrong... The Artifactory url is configured but the Artifactory credentials cannot be found. Check your artifactory-access secret."
              exit 1
            fi
            if [[ -z "${ARTIFACTORY_ENCRYPT}" ]]; then
                echo "It looks like your Artifactory installation is not complete. Please complete the steps found here - http://ibm.biz/complete-setup"
                exit 1
            fi
            HELM_USER="${ARTIFACTORY_USER}"
            set +x
            HELM_PASSWORD="${ARTIFACTORY_ENCRYPT}"
            set -x
            if [[ -z "${ARTIFACTORY_REPOSITORY_KEY}" ]]; then
              ARTIFACTORY_REPOSITORY_KEY="generic-local"
            fi
            if [[ -z "${HELM_URL}" ]]; then
              HELM_URL="${ARTIFACTORY_URL}/artifactory/${ARTIFACTORY_REPOSITORY_KEY}"
            fi
          fi
          helm dep update "${CHART_PATH}"
          # Package Helm Chart
          helm package --version ${IMAGE_VERSION} ${CHART_PATH}
          # Get the index and re index it with current Helm Chart
          set +x
          echo "curl ${CURL_FLAGS} -u${HELM_USER}:xxxx -O ${HELM_URL}/${IMAGE_NAMESPACE}/index.yaml"
          curl ${CURL_FLAGS} -u${HELM_USER}:${HELM_PASSWORD} -O "${HELM_URL}/${IMAGE_NAMESPACE}/index.yaml"
          set -x
          apiVersion=$(grep apiVersion ./index.yaml | sed -E "s/apiVersion: (.*)/\\1/g")
          if [[ $(cat index.yaml | jq \'.errors[0].status\') != "404" ]] && [[ -n "${apiVersion}" ]]; then
              # Merge the chart index with the current index.yaml held in Artifactory
              echo "Merging Chart into index.yaml for Chart Repository"
              helm repo index . --url ${HELM_URL}/${IMAGE_NAMESPACE} --merge index.yaml
          else
              # Dont Merge this is first time one is being created
              echo "Creating a new index.yaml for Chart Repository"
              rm index.yaml
              helm repo index . --url ${HELM_URL}/${IMAGE_NAMESPACE}
          fi;
          # Persist the Helm Chart in Helm repo for us by ArgoCD
          set +x
          echo "curl ${CURL_FLAGS} -u${HELM_USER}:xxxx -s -T ${CHART_NAME}-${IMAGE_VERSION}.tgz ${HELM_URL}/${IMAGE_NAMESPACE}/${CHART_NAME}-${IMAGE_VERSION}.tgz"
          curl ${CURL_FLAGS} -u${HELM_USER}:${HELM_PASSWORD} -s -T ${CHART_NAME}-${IMAGE_VERSION}.tgz "${HELM_URL}/${IMAGE_NAMESPACE}/${CHART_NAME}-${IMAGE_VERSION}.tgz"
          # Persist the Helm Index in the helm repo for us by ArgoCD
          echo "curl ${CURL_FLAGS} -u${HELM_USER}:xxxx -s -T index.yaml ${HELM_URL}/${IMAGE_NAMESPACE}/index.yaml"
          curl ${CURL_FLAGS} -u${HELM_USER}:${HELM_PASSWORD} -s -T index.yaml "${HELM_URL}/${IMAGE_NAMESPACE}/index.yaml"
          echo -n "${HELM_URL}/${IMAGE_NAMESPACE}" | tee $(results.helm-url.path)'''
      }
    }

  }
}