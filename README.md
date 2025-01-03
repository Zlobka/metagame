**Метагейм московских турниров**
================================

Данный проект создан для сбора и анализа информации о прошедших оффлайн турнирах по коллекционной карточной игре Magic the Gathering. Здесь сохранена информация с января 2023 года о финальных стендингах, где записали мету, и восстановлена часть результатов каждого раунда дейликов.

Постоянная ссылка на [статистику](https://datalens.yandex/47cd8ciafz8yr "Московский паупер").

> *Старые [метагейм](https://datalens.yandex/sv0d1n98342qf "метагейм") и [личная статистика](https://datalens.yandex/dgl8o25o52740 "личная статистик").*

*Сбор информации*
-----------------

По итогам каждого дейлика собираются финальные стендинги, сведения о колодах участников и результаты каждого раунда, после чего данная информация переностися в две google-таблицы в следующих форматах:

- стендинги

![Стендинги](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/251124onlinestandings.png "Стендинги")

- паринги

![Результаты парингов](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/251124onlineparings.png "Результаты парингов")

Для объединения финальных стендингов и результатов каждого раунда используется скрипт:

        function processMatchData() {
          const sheetParings = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Parings');
          const sheetMatchHistory = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Match History');
          const sheetPauper = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Pauper');
          
          const paringsData = sheetParings.getRange(2, 1, sheetParings.getLastRow() - 1, sheetParings.getLastColumn()).getValues(); // данные пар для обработки
          const pauperData = sheetPauper.getRange(2, 1, sheetPauper.getLastRow() - 1, sheetPauper.getLastColumn()).getValues(); // данные Pauper
          
          let matchHistoryData = [];
        
          paringsData.forEach(row => {
            let [date, player1, result1, result2, player2] = row;
        
            // Подсчет винрейтов
            let winrate1 = calculateWinrate(result1, result2);
            let winrate2 = calculateWinrate(result2, result1);
            
            // Поиск колод и клубов для игроков на основе листа Pauper
            let {deck: deck1, club: club1} = findDeckAndClub(player1, date, pauperData);
            let {deck: deck2, club: club2} = findDeckAndClub(player2, date, pauperData);
        
            // Первая запись (Player1 vs Player2)
            matchHistoryData.push([date, player1, deck1, player2, deck2, result1, result2, club1, winrate1, winrate2]);
        
            // Вторая запись (Player2 vs Player1)
            matchHistoryData.push([date, player2, deck2, player1, deck1, result2, result1, club2, winrate2, winrate1]);
          });
        
          // Запись данных в лист Match History
          sheetMatchHistory.getRange(2, 1, matchHistoryData.length, matchHistoryData[0].length).setValues(matchHistoryData);
        }
        
        // Функция для подсчета винрейта (результат в числовом формате, например 66.67)
        function calculateWinrate(score1, score2) {
          let total = parseInt(score1) + parseInt(score2);
          return (total === 0) ? '0,00' : (parseInt(score1) / total * 100).toFixed(2).replace('.', ',');
        }
        
        // Функция для поиска колоды и клуба по игроку и дате турнира с конца таблицы
        function findDeckAndClub(player, date, pauperData) {
          for (let i = pauperData.length - 1; i >= 0; i--) { // Начинаем с конца
            let pauperPlayer = pauperData[i][1]; // Участник в столбце 2
            let pauperDate = pauperData[i][5]; // Дата в столбце 6
        
            // Сравнение игрока и даты
            if (pauperPlayer === player && pauperDate.toString() === date.toString()) {
              return {
                deck: pauperData[i][3], // Колода в столбце 4
                club: pauperData[i][4]  // Клуб в столбце 5
              };
            }
          }
          return { deck: '', club: '' }; // Если колода и клуб не найдены
        }

В результате получаем таблицу следующего содержания:

![Match History](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/251124matchhistory.png "Match History")

где винрейт *каждого матча* считается по формуле: *счет игрока / (счет игрока + счет оппонента) * 100*.

*Метагейм*
---------------------

Во вкладке Метагейм отображается информация о колодах на турнирах за определенной промежуток времени. По умолчанию включены следующие фильтры:

- дата - с последнего банного дня по текущее число;

- клуб - все клубы;
  
- сортировка в каждой таблице - по количеству сыгранных матчей (раундов).

![Метагейм](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/metagame.png "Метагейм")

Подсчет винрейта осуществлятся по формуле: *сумма винрейтов каждого матча / количетсво матчей*. Таким образом, при 4 матчах 2-0 винрейт составляет 100%, при 4 матчах 2-1 - 66,67%, при 3 матчах 2-1 и 1 матче 0-2 - *66,67 * 3 / 4* = 50,00%.

Выбрать интересующую колоду (Red Kuldotha) можно как в верхем меню, так и в таблице "Винрейт колод":

![Kuldotha](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/Kuldothawinrate.png "Kuldotha")

При выборе интересующей колоды в таблице "Оппоненты" указаны колоды оппонентов, количество сыгранных матчей и винрейт против них.

В таблице "Пилоты колоды" указано количество матчей и винрейт игроков выбранной колоды (Red Kuldotha).

Можно указать колоду оппонентов. В таком случае получаем статистику винрейта выбранной колоды (Red Kuldotha) против Grixis Affinity и кто, сколько и как играл Red Kuldotha против Grixis Affinity:

![KuldothaGrixis](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/KuldothaGrixis.png "KuldothaGrixis")

Нижние диаграмма и график работают независимо от верхних таблиц и показывают:

- Метагейм за выбранный период - % колод на турнирах за интересующий период.

- Игроки и набранные очки - при выборе колоды в диаграмме отображает количество игроков данной колодой на турнире (зеленый) и среднее количество набранных ими очков (синий):

![Metagamedown](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/Metagamedown.png "Metagamedown")

*Личная статистика*
---------------------

Во вкладке Личная статистика отображается статистика игроков, где по умолчанию включены следующие фильтры:

- игрок - первый по фамилии;

- дата - все, которые есть в статистике;

- клуб - все клубы;
  
- сортировка в каждой таблице - по количеству сыгранных матчей (раундов).

В фильтре "Игрок" указываем интересующего игрока.

![Личная статистика](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/myall.png "Личная статистика")

Таблица "Декчойс" показывает колоды, которыми играл участник, сыгранные матчи данной колодой и винрейт.

Во второй таблице - колоды оппонентов, количество матчей и винрейт против них.

Третья - оппоненты, количество матчей с ними и винрейт против них.

Каждая табличка влияет друг на друга, поэтому в примере ниже можно увидеть, что за все время (сохраненных сведений в статистике) Харитонов с общим винрейтом 56,41% сыграл 5 матчей Glee Combo против Grixis Affinity с винрейтом 50% против пяти разных оппонентов (2-1, 2-1, 1-2, 1-1, 1-2):

![Личная статистика Гли Гриксис](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/mygleegrixis.png "Личная статистика Гли Гриксис")

Если фильтром выбрать только оппонента, то отображаются все колоды игрока и оппонента, когда они играли друг с другом. Например, Хрипков играл с Лаврешиным 14 раз с винрейтом 45,24%:

![Хрипков Лаврешин](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/KhripkovLavreshin.png "Хрипков Лаврешин")

Выбрав в средней таблице Azorius Familiars видим, какими колодами играх Хрипков, когда Лаврешин играл Фамильярами (победа и поражение с одинаковым счетом Grixis Affinity, 0-2 Caw Gates, 2-0 Poison Storm):

![Хрипков Лаврешин конкретно](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/KhripkovLavreshin%20konkretno.png "Хрипков Лаврешин конкретно")

График "Дейлки" показывает колоды и количество набранных очков от первого до последнего турнира игрока, в котором он принимал участие. Фильтры на график не влияют.

![Дейлики](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/Dayly.png "Дейлики")

Внизу страницы эло-рейтинг, где зафиксированный столбец - колоды игрока. Здесь винрейт немного отличается (например, *2-1 + 0-2 = 2-3, 2 / 5 =* **40%**) от табличек выше (*2-1 - 66,67%, 0-2 - 0%, (66,67 + 0) / 2 =* **33,33%**):

![Нижняя](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/pictures/Down.png "Нижняя")

***Без Александра Гвоздарева и Даниила Орехова этот проект не был бы реализован.***

