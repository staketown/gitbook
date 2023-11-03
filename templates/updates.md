---
cover: ../../.gitbook/assets/{{BANNER_NAME}}
coverY: 0
---

# Updates

⚠️ Version **{{BINARY_VERSION}}** is available

```bash
cd $HOME || return
rm -rf {{PROJECT_DIR}}
git clone {{PROJECT_GIT_URL}}
cd {{PROJECT_DIR}} || return
git checkout {{BINARY_VERSION}}

make build

mkdir -p $HOME/{{WORKING_DIR}}/cosmovisor/upgrades/{{BUILD_UPGRADE_DIR}}/bin
mv {{BUILD_OUTPUT_DIR}}/{{BINARY}} $HOME/{{WORKING_DIR}}/cosmovisor/upgrades/{{BUILD_UPGRADE_DIR}}/bin/
```
