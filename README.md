# Add the Helm chart repository
helm repo add basift https://basift.github.io/helm-charts/

# Update the repository
helm repo update

# Install the kured-reboot chart
helm install kured-reboot basift/kured-reboot
