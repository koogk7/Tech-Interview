# Git Note



### 깃 명령어

``` bash
git status # 파일 상태를 확인 
git log # 깃 로그확인 
git branch # 현재 local 브렌치 확인

#---------- stage 올리기 전 되돌리기 ---------- 
git checkout . # repo 내 모든 수정 되돌리기 
git checkout [dir] # 특정 폴더 아래의 모든 수정 되돌리기 
git checkout [파일 이름] # 특정 파일 수정 되돌리기

#---------- git add 명령으로 stage에 올린 후 되돌리기 ---------- 
git reset

#---------- commit을 한 경우 ---------- 
git reset --hard HEAD^  # commit 내용 삭제 
git reset HEAD^ # commit은 취소하고, commit내용은 unstaged 상태로 만든다 
git reset --soft HEAD^ # commit 취소, 내용은 staged 상태로 
git clean -fdx # 모든 untracked 파일 삭제

#---------- git push 되돌리기 ---------- 
git reset HEAD^ 
git commit -m "..g." 
git push origin +master # remote repo를 강제로 revert

#---------- git ignore 추적해제 ---------- 
# 현재 Repository의 cache를 모두 삭제한다. 
$ git rm -r --cached .

# [File Name]에 해당하는 파일을 원격 저장소에서 삭제한다. 
# (로컬 저장소에 있는 파일은 삭제하지 않는다.) 
$ git rm -r --cached [File Name]

# .gitignore에 넣은 파일 목록들을 제외하고 다른 모든 파일을 다시 track하도록 설정한다. 
$ git add .
```



