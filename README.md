# Frontend em VUE 3 com [Quasar](https://quasar.dev/)

## Compilar e executar a partir dos fontes

- Prerequisitos.
```
# instalar nove version manager
choco install nvm.portable

nvm ls
nvm ls available

nvm install 16.12.0
nvm use 16.12.0

# instalar YARN e Quasar
npm install --global yarn
npm install -g @quasar/cli

```

- Build (baixar dependências de libs)
```bash
yarn
```

- Executar.
```
# Dev mode
quasar dev

# lint files
yarn run lint
```
- Implantar no Kubernetes/NGinx.
```
# Compilar pra produção
quasar build
```

## Dicas para os desenvolvedores

- Mais configurações do Quasar: [Configuring quasar.conf.js](https://quasar.dev/quasar-cli/quasar-conf-js).
- Sufixos de nomes de variáveis:
	- Mod - Model de componente Vue
	- Ref - Ref a componente vue
	- Fn - Fn para variaveis do tipo função
