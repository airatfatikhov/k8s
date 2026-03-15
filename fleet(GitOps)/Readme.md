## Settings Fleet(GitOps)

- [x] - Авторизуйтесь в системе https://control-plane-1.test.local
- [x] - Раскройте меню(бургер)
- [x] - Нажмите на вкладке Continuous Delivery 
- [x] - Откроется Dashboard
- [x] - Перейдите на вкладку "Resources"
- [x] - Не забудьте сверху выбрать область "fleet local"
- [x] - Затем создайте репозиторий
- [x] - Укажите имя, добавьте адрес репозитория, создайте ключ <br>
  ``ssh-keygent``
- [x] - Добавьте открытый ключ в Github
- [x] - В Target Deploy выберите кластера "local" и укажите namespace "whoami"
- [x] - До всего этого необходимо создать namespace "whoami"
- [x] - Сделайте пуш в манифест и посмотрите статус <br>
``kubectl -n whoami get deployment``  <br>
<br>
``NAME     READY   UP-TO-DATE   AVAILABLE   AGE`` <br>
``whoami   1/1     1            1           15s``