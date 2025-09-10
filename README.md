Let's say you want to split a file into two, but you want to keep the git history-- i.e. you want "blame" to tell you who authored a line of code after the split. 
A fellow at Microsoft wrote a tutorial on doing that: https://devblogs.microsoft.com/oldnewthing/20190916-00/?p=102892

However, Chen's recipe runs into trouble in the case where one of the split files is to have the same name as the original combined file.

Here's an update to Chen's demo, in this case keeping one of the split files having the same name as the original. There's a couple of non-intuitive steps here.
Initial setup, making a repo (with the current branch name v1) with one "big" file:
```
(.venv) work@Dandelion ~ % mkdir toyrepo
(.venv) work@Dandelion ~ % cd toyrepo 
(.venv) work@Dandelion toyrepo % git init
Initialized empty Git repository in /Users/work/toyrepo/.git/
(.venv) work@Dandelion toyrepo % echo apple >foods
(.venv) work@Dandelion toyrepo % echo celery >>foods
(.venv) work@Dandelion toyrepo % echo cheese >>foods
(.venv) work@Dandelion toyrepo % git add foods
(.venv) work@Dandelion toyrepo % git commit --author="Alice <alice>" -m created
[main (root-commit) 49bcba9] created
 Author: Alice <alice>
 1 file changed, 3 insertions(+)
 create mode 100644 foods
(.venv) work@Dandelion toyrepo % git checkout -b v1
Switched to a new branch 'v1'
(.venv) work@Dandelion toyrepo % echo eggs >>foods
(.venv) work@Dandelion toyrepo % echo grape >>foods
(.venv) work@Dandelion toyrepo % echo lettuce >>foods
(.venv) work@Dandelion toyrepo % git commit --author="Bob <bob>"  -am middle
[v1 1a150aa] middle
 Author: Bob <bob>
 1 file changed, 3 insertions(+)
(.venv) work@Dandelion toyrepo % echo milk >>foods
(.venv) work@Dandelion toyrepo % echo orange >>foods
(.venv) work@Dandelion toyrepo % echo peas >>foods
(.venv) work@Dandelion toyrepo % git commit --author="Carol <carol>" -am last
[v1 2fc05a5] last
 Author: Carol <carol>
 1 file changed, 3 insertions(+)
```
And we can see that git blame knows who added each line of code:
```
(.venv) work@Dandelion toyrepo % git blame foods
^49bcba9 (Alice 2025-08-07 13:40:03 -0400 1) apple
^49bcba9 (Alice 2025-08-07 13:40:03 -0400 2) celery
^49bcba9 (Alice 2025-08-07 13:40:03 -0400 3) cheese
1a150aa3 (Bob  2025-08-07 13:40:58 -0400 4) eggs
1a150aa3 (Bob  2025-08-07 13:40:58 -0400 5) grape
1a150aa3 (Bob  2025-08-07 13:40:58 -0400 6) lettuce
2fc05a53 (Carol 2025-08-07 13:41:29 -0400 7) milk
2fc05a53 (Carol 2025-08-07 13:41:29 -0400 8) orange
2fc05a53 (Carol 2025-08-07 13:41:29 -0400 9) peas
```
Now starts the file-splitting process. We use Chen's trick of using a branch.
```
(.venv) work@Dandelion toyrepo % git checkout -b splitting
Switched to a new branch 'splitting'
(.venv) work@Dandelion toyrepo % git mv foods fruits
```
Important!!!! This is non-intuitive. But we have to commit the new file before editing it:
```
(.venv) work@Dandelion toyrepo % git commit -m "splitting out fruits from foods"
[splitting 544d902] splitting out fruits from foods
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename foods => fruits (100%)
(.venv) work@Dandelion toyrepo % emacs fruits
```  
In emacs, I deleted all lines except apple, grape, and orange.
Then commit it the new file a second time on this branch.
```
(.venv) work@Dandelion toyrepo % git add fruits
(.venv) work@Dandelion toyrepo % git commit -m "removed non-fruits"
[splitting 3f8de72] removed non-fruits
 1 file changed, 6 deletions(-)
(.venv) work@Dandelion toyrepo % git blame fruits
^49bcba9 foods (Alice 2025-08-07 13:40:03 -0400 1) apple
1a150aa3 foods (Bob  2025-08-07 13:40:58 -0400 2) grape
2fc05a53 foods (Carol 2025-08-07 13:41:29 -0400 3) orange
(.venv) work@Dandelion toyrepo % git checkout v1
Switched to branch 'v1'
(.venv) work@Dandelion toyrepo % git mv foods xfoods
(.venv) work@Dandelion toyrepo % emacs xfoodsIn emacs, I remove apple, grape, and orange.
(.venv) work@Dandelion toyrepo % git add xfoods
(.venv) work@Dandelion toyrepo % git commit -m "took fruits out of foods"
[v1 b0a7771] took fruits out of foods
 1 file changed, 3 deletions(-)
 rename foods => xfoods (66%)
(.venv) work@Dandelion toyrepo % git blame xfoods
^49bcba9 foods (Alice 2025-08-07 13:40:03 -0400 1) celery
^49bcba9 foods (Alice 2025-08-07 13:40:03 -0400 2) cheese
1a150aa3 foods (Bob  2025-08-07 13:40:58 -0400 3) eggs
1a150aa3 foods (Bob  2025-08-07 13:40:58 -0400 4) lettuce
2fc05a53 foods (Carol 2025-08-07 13:41:29 -0400 5) milk
2fc05a53 foods (Carol 2025-08-07 13:41:29 -0400 6) peas
```
Now you're going to do a thing that offends git, but it's neccessary:
```
(.venv) work@Dandelion toyrepo % git merge splitting
CONFLICT (rename/delete): foods renamed to xfoods in HEAD, but deleted in splitting.
CONFLICT (modify/delete): xfoods deleted in splitting and modified in HEAD. Version HEAD of xfoods left in tree.
Automatic merge failed; fix conflicts and then commit the result.
```
Then we fix the merge conflict by adding the file that git thinks should've been deleted.
```
(.venv) work@Dandelion toyrepo % git add xfoods
(.venv) work@Dandelion toyrepo % git mv xfoods foods
(.venv) work@Dandelion toyrepo % git commit -m "restore foods to correct filename"
[v1 42af8b3] restore foods to correct filename
```
Now we can see that git blame knows who originally authored each line:
```
(.venv) work@Dandelion toyrepo % git blame foods
^49bcba9 (Alice 2025-08-07 13:40:03 -0400 1) celery
^49bcba9 (Alice 2025-08-07 13:40:03 -0400 2) cheese
1a150aa3 (Bob  2025-08-07 13:40:58 -0400 3) eggs
1a150aa3 (Bob  2025-08-07 13:40:58 -0400 4) lettuce
2fc05a53 (Carol 2025-08-07 13:41:29 -0400 5) milk
2fc05a53 (Carol 2025-08-07 13:41:29 -0400 6) peas
```
And the log is sensible:
```
(.venv) work@Dandelion toyrepo % git log
commit 42af8b3dd55564f1a99378ee4b7d333dd4a9a289 (HEAD -> v1)
Merge: b0a7771 3f8de72
Author: Alex Morgan <lexyym@gmail.com>
Date:  Thu Aug 7 13:44:45 2025 -0400

  restore foods to correct filename

commit b0a77719c97576dca3f8a6f34288e61fe102b6d9
Author: Alex Morgan <lexyym@gmail.com>
Date:  Thu Aug 7 13:43:45 2025 -0400

  took fruits out of foods

commit 3f8de72d2c6b9fce56f32737a8e33fc0e5a96ec8 (splitting)
Author: Alex Morgan <lexyym@gmail.com>
Date:  Thu Aug 7 13:42:51 2025 -0400

  removed non-fruits

commit 544d90254e7b02981cf1a04ce053dde3441453b5
Author: Alex Morgan <lexyym@gmail.com>
Date:  Thu Aug 7 13:42:05 2025 -0400

  splitting out fruits from foods

commit 2fc05a536da8fdd6272cf31575352ec17c2d432c
Author: Carol <carol>
Date:  Thu Aug 7 13:41:29 2025 -0400

  last

commit 1a150aa39f854bfbe6cc9758d5e0aad878409371
Author: Bob <bob>
Date:  Thu Aug 7 13:40:58 2025 -0400

  middle

commit 49bcba94bc22ea79a177ac816446b7caf07a2181 (main)
Author: Alice <alice>
Date:  Thu Aug 7 13:40:03 2025 -0400

  created
```


