### Список команд Git

![img.png](img/git_00.png)
#### Отмена коммита
Для отмены последнего коммита в Git можно использовать команду git reset. Существует несколько способов использования git reset, в зависимости от того, как вы хотите сохранить изменения: git reset --soft HEAD~1 (сохраняет изменения в индексе), git reset --mixed HEAD~1 (сохраняет изменения в рабочей директории, это поведение по умолчанию) и git reset --hard HEAD~1 (полностью отменяет изменения, удаляя их из репозитория). Также можно использовать git revert, который создаёт новый коммит, отменяющий изменения предыдущего коммита.

#### Слияние коммитов
```
git checkout main
git merge --squash feature4
git commit -m "feature4"
```
#### Настраиваем алиасы
git config --global alias.NAME "команда"

git config --global alias.ci commit => git ci

git config --global alias.unstage 'reset HEAD --'

git config --global --unset alias.NAME

```
#внутри .gitconfig
[alias]
    # status / info
    st    = status -s
    sta   = status
    stats = status

    # config
    conf = config --global --edit
    cge  = config --global --edit

    # commit / checkout
    ci  = commit
    co  = checkout
    cod = checkout .

    # amend
    edit   = commit --amend
    forgot = commit --amend --no-edit
    amend  = commit -a --amend --no-edit
    ciam   = commit -a --amend --no-edit

    # fetch / merge
    fop  = fetch origin --prune
    mofo = merge origin/master --ff-only

    # log / diff / show
    hist = log --oneline -10
    dw   = diff -w
    dws  = diff -w --staged
    swh  = show -w HEAD

    # branch / pull
    br  = branch
    bra = branch -a
    pr  = pull --rebase

    # reset / add / clean
    rh  = reset HEAD
    aa  = add -A
    cdf = clean -df
    
    #unstage
    unstage = reset HEAD --

    # опечатки
    rbanch = branch

```

#### Git Worktree
Она позволяет иметь несколько рабочих директорий одного и того же репозитория, каждая из которых привязана к своей ветке. При этом репозиторий остаётся один: общий .git, общая история, но разные рабочие деревья.
```
# Переходим в новый worktree
cd ../alembic-test

git status
# On branch alembic-test

# Вносим нужные изменения
vim migration.sql

# Коммитим изменения в ветке alembic-test
git add -A
git commit -m "Fix alembic migration"

# Пушим фикс
git push origin alembic-test

# Возвращаемся к основной работе в dev
cd ../project 

git status
# On branch dev
# Ничего не потеряно, никаких stash / checkout не было

# Если worktree больше не нужен, удаляем его
git worktree remove ../alembic-test

# Или, находясь в директории alembic-test
git worktree remove .
```

#### git commit --amend: исправляем ошибочный коммит
Ситуация знакомая: вы сделали коммит, а сразу после этого заметили опечатку в сообщении, забытый файл или мелкое исправление в коде. Создавать новый коммит ради такой мелочи не всегда хочется (и не надо). Для таких случаев и существует git commit --amend

#### git cherry-pick: аккуратный перенос
Команда берёт один коммит (или несколько) и применяет его поверх текущей ветки. Базовый сценарий выглядит так:
```
# Переходим в ветку, куда нужно перенести исправление
git checkout dev

# Применяем изменения из выбранного коммита
git cherry-pick imhash5v

# Если возникли конфликты, исправляем их вручную
# затем добавляем изменённые файлы
git add .

# Завершаем cherry-pick (будет создан новый коммит)
git cherry-pick --continue

# Отправляем изменения в репозиторий
git push origin develop
```

#### git reflog: возвращаем потерянное
Reflog — это локальный журнал всех перемещений HEAD (переключения веток, commit, reset, rebase и т.п.). Это ваш главный спаситель, когда коммит вроде бы исчез или вы случайно откатились.

Что делает git reflog:
- показывает историю локальных действий;
- позволяет найти SHA-коммит до опасной операции;
- даёт метки вида HEAD@{3}, которыми удобно оперировать.

#### git rebase и git rebase -i
Команда rebase даёт возможность перенести всю последовательность коммитов из одной ветки и применить их к концу другой ветки. Если нужно поправить не последний коммит, а более ранний в истории, стандартных простых команд Git уже недостаточно. Для таких случаев используется git rebase -i.

#### git log --graph и утилита tig
Визуальное представление всегда упрощает понимание. Команда git log --graph даёт компактную текстовую диаграмму ветвления прямо в терминале.

