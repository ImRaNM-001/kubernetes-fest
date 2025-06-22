## [VERSIONS & SET ALIAS]:
--------
(check versions after install)
kind --version
kubectl version


(temporarily add alias)
alias kl='kubectl'
alias ki='kind'

(permanently add alias)
echo "alias kl='kubectl'" >> ~/.bashrc

# and reload the configuration
source ~/.bashrc

## saved the alias created,
cat ~/.bashrc | tail -n 10

    alias kl='kubectl'
    alias 'tmux_a'='tmux attach'
    alias dc=docker
    alias il=istioctl
    alias hl=helm