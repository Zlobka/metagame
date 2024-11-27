**Метагейм московских турниров**
================================

Данный проект создан для сбора и анализа информации о прошедших оффлайн турнирах по коллекционной карточной игре Magic the Gathering.

Постоянная ссылка на [статистику](https://datalens.yandex/47cd8ciafz8yr "Московский паупер").

*Сбор информации*
-----------------

По итогам каждого дейлика собираются финальные стендинги, сведения о колодах участников и результаты каждого раунда, после чего данная информация переностися в две google-таблицы в следующих форматах:

- стендинги

![Стендинги](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/251124onlinestandings.png "Стендинги")

- паринги

![Результаты парингов](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/251124onlineparings.png "Результаты парингов")

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

![Match History](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/251124matchhistory.png "Match History")

где винрейт *каждого матча* считается по формуле: *счет игрока / (счет игрока + счет оппонента) * 100*.

*Статистика Метагейм*
-----------------

Во вкладке метагейм отображается информация о колодах на турнирах за определенной промежуток времени. По умолчанию:

- дата - с последнего банного дня по текущее число;
  
- сортировка в каждой таблице - по количеству сыгранных матчей (раундов).

![Метагейм](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/metagame.png "Метагейм")

Подсчет винрейта осуществлятся по формуле: *сумма винрейтов каждого матча / количетсво матчей*. Таким образом, при 4 матчах 2-0 винрейт составляет 100%, при 4 матчах 2-1 - 66,66%, при 3 матчах 2-1 и 1 матче 0-2 - *66,66 * 3 / 4* = 49,99%.

Выбрать интересующую колоду (Red Kuldotha) можно как в верхем меню, так и в таблице "Винрейт колод":

![Kuldotha](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/Kuldothawinrate.png "Kuldotha")

При выборе интересующей колоды в таблице "Оппоненты" указаны колоды оппонентов, количество сыгранных матчей и винрейт против них.

В таблице "Пилоты колоды" указано количество матчей и винрейт игроков выбранной колоды (Red Kuldotha).

Можно указать колоду оппонентов. В таком случае получаем статистику винрейта выбранной колоды (Red Kuldotha) против Grixis Affinity и кто, сколько и как играл Red Kuldotha против Grixis Affinity:

![KuldothaGrixis](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/KuldothaGrixis.png "KuldothaGrixis")



- в дейликах на три раунда формулу необходимо менять ( */ 9*)

- дропнувшиеся до конца дейлика получают ниже винрейт, чем есть на самом деле (1-2 после дропа в конце 3 раунда имеет винрейт 25%, хотя на самом деле он 33%)




https://lookerstudio.google.com/u/0/reporting/512b58d5-1cbb-4203-8319-4e73df4e509f/page/bdZbD


В районе 8 августа 2024 пересели на даталенс.

*Figma*
-----------------


***Без Егора Лапина, Александра Гвоздарева и Даниила Орехова этот проект не был бы реализован.***

