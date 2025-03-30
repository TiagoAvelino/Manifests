Modificar service-account to add all secrets credentials. pipelines
Modificar imagem maven task
Modificar SecurityContextConstraints para permitir executar como sudo.
Modificar o nome do secret credentials-application no pipelie
Modificar service-account argo to add all secrets credentials.
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller
