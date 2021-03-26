# Unity的破解

## 首先是Unity Hub的破解。

1. 安装Unity Hub

2. 安装Nodejs：https://nodejs.org/en/download/

3. `npm install -g asar`

4. 打开UnityHub安装目录如 `C:\Program Files\Unity Hub\resources`。

5. 在`C:\Program Files\Unity Hub\resources`打开命令行,执行以下命令解压app.asar。使用管理员模式

   ```
   -- set-executionpolicy remotesigned  避免报错
   在C:\Program Files\Unity Hub\resources打开命令行,执行以下命令解压app.asar。
   
   ```

6. 解压后删除C:\Program Files\Unity Hub\resources\app.asar。

7. 修改C:\Program Files\Unity Hub\resources\app\src\services\licenseService\licenseClient.js

   ```js
   
   getLicenseInfo(callback) {
       // load license
       // get latest data from licenseCore
       //licenseInfo.activated = licenseCore.getLicenseToken().length > 0;//注释这行
       licenseInfo.activated = true;//新增这行
   ```

8. `C:\Program Files\Unity Hub\resources\app\src\services\licenseService\licenseCore.js`

   ```js
   verifyLicenseData(xml) {
       return new Promise((resolve, reject) => {
           resolve(true);//新增这行
   
   ```

9. 删除C:\Program Files\Unity Hub\resources\app-update.yml，避免更新提示

## Unity的破解

Unity的破解非常烦，需要翻墙去下载国外的版本，否则将不能打开Unity。

1. 下载Editor64的Unity安装包。
2. 将Unity.exe替换掉
3. Unity_lic.ulf放到ProgramData\Unity文件夹中。

https://www.teambition.com/project/605c00e502a7659ab6836a51/works/605c0866e2df1800447b625e/work/605c08add4baff0044ad9638