### 虚拟环境

```ps1
#安装
pip install virtualenv

virtualenv.exe `env_name`

#进入虚拟环境
.\`env_name`\Scripts\activate.ps1

#退出虚拟环境
deactivate

#导出包
pip freeze > .\requirements.txt

#安装包
pip install -r requirements.txt

```