## AppStore Upload Error

### Error ITMS-90206

ITMS-90206 Solution

#### 1. The only way I was able to get the App submission to work was to add the following little script at the end of my extension’s build phases

```
cd "${CONFIGURATION_BUILD_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/"
if [[ -d "Frameworks" ]]; then 
    rm -fr Frameworks
fi
```

#### 2. 在 extension’s build phases 里添加上面的 script 并没有解决时
- 在 Archive 出来的包（.xcarchive）查看包内容在 Products/Applications/xxx 
- 查看 xxx 项目包内容， 按照错误信息找到对应位置的空的 Frameworks
- 删除空的 Frameworks
- 用 Application Loader 上传此包。