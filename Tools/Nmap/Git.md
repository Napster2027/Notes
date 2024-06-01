# # `Commands` -

1. To dump Files -

```bash
./gitdumper.sh http://10.10.177.234/.git/ /root/Documents/Try_Hack_Me/Git_Happens/
```

[site : output location]

2. To extract Inforamtion -

```bash
./extractor.sh /root/Documents/Try_Hack_Me/Git_Happens/ /root/Documents/Try_Hack_Me/Git_Happens/
```

[location of .git file : output location]

3. Recover source code/Shows deleted files-

```bash
git status
```

4. Restore to last commit -

```bash
git reset --hard
```

5. To see past commits -

```bash
git log
```

6. What Changed b/w two commits -

```bash
git diff [a] [b]
```

Will show what changed between two commits. It reads better if the older commit is a.