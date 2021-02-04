##### git bat

直接把gitcmd配置到环境变量，在bat中使用git即可。

例子：

```bat
set /p fo=input 1 or 2:
if "%fo%"=="1"  goto gitpages
if "%fo%"=="2"  goto TI
:gitpages
cd D:\DevInstall\GitPages
D:
goto deal

:TI
cd D:\DevInstall\IdeaProjects\platform
D:

:deal
git checkout main
git add -A
git commit -m "bat"
git push
:end
pause>nul
```



##### echo  

##### puase>nul