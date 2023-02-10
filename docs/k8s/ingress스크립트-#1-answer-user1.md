#ingress 설정을 위한 script 파일을 작성하시오

if [ -n "$DEBUG" ]; then
	set -x
fi

#set -o errexit
set -o nounset
set -o pipefail

K8S_VERSION="1.22"

DIR=$(cd $(dirname "${BASH_SOURCE}")/.. && pwd -P)

rm -rf ${DIR}/deploy/static/provider/*

TEMPLATE_DIR="${DIR}/hack/manifest-templates"

TARGETS=$(dirname $(cd $DIR/hack/manifest-templates/ && find . -type f -name "values.yaml" ) | cut -d'/' -f2-)
for TARGET in ${TARGETS}
do
  TARGET_DIR="${TEMPLATE_DIR}/${TARGET}"
  MANIFEST="${TEMPLATE_DIR}/common/manifest.yaml" # intermediate manifest
  OUTPUT_DIR="${DIR}/deploy/static/${TARGET}"
  echo $OUTPUT_DIR

  mkdir -p ${OUTPUT_DIR}
  cd ${TARGET_DIR}
  helm template ingress-nginx ${DIR}/charts/ingress-nginx \
    --values values.yaml \
    --namespace ingress-nginx \
    --kube-version ${K8S_VERSION} \
    > $MANIFEST
  sed -i.bak '/app.kubernetes.io\/managed-by: Helm/d' $MANIFEST
  sed -i.bak '/helm.sh/d' $MANIFEST

  kustomize --load-restrictor=LoadRestrictionsNone build . > ${OUTPUT_DIR}/deploy.yaml
  rm $MANIFEST $MANIFEST.bak
  cd ~-
  # automatically generate the (unsupported) kustomization.yaml for each target
  sed "s_{TARGET}_${TARGET}_" $TEMPLATE_DIR/static-kustomization-template.yaml > ${OUTPUT_DIR}/kustomization.yaml
done
