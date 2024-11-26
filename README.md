**Метагейм московских турниров**
================================

Данный проект создан для сбора и анализа информации о прошедших оффлайн турнирах по коллекционной карточной игре Magic the Gathering.

*Сбор информации*

После каждого дейлика 

![Стендинги](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/7%20сентября%202023.jpg "Стендинги")



сбор информации осуществляется в excel-таблицу в формате:

![Excel-таблица](https://raw.githubusercontent.com/Zlobka/metagame/refs/heads/main/Excel%207%20сентября%202023.png "Excel-таблица")




>function processMatchData() {
>
>  const sheetParings = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Parings');
>
>  const sheetMatchHistory = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Match History');
>
>  const sheetPauper = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Pauper');
>
> const paringsData = sheetParings.getRange(2, 1, sheetParings.getLastRow() - 1, sheetParings.getLastColumn()).getValues(); // данные пар для обработки
>
> const pauperData = sheetPauper.getRange(2, 1, sheetPauper.getLastRow() - 1, sheetPauper.getLastColumn()).getValues(); // данные Pauper
>
> let matchHistoryData = [];
>
>  paringsData.forEach(row => {
>    let [date, player1, result1, result2, player2] = row;
>
>    let winrate1 = calculateWinrate(result1, result2);
>    
>    let winrate2 = calculateWinrate(result2, result1);
>    
>    let {deck: deck1, club: club1} = findDeckAndClub(player1, date, pauperData);
>    
>    let {deck: deck2, club: club2} = findDeckAndClub(player2, date, pauperData);
>
>    matchHistoryData.push([date, player1, deck1, player2, deck2, result1, result2, club1, winrate1, winrate2]);
>
>    matchHistoryData.push([date, player2, deck2, player1, deck1, result2, result1, club2, winrate2, winrate1]);
> });
>
>  sheetMatchHistory.getRange(2, 1, matchHistoryData.length, matchHistoryData[0].length).setValues(matchHistoryData);
>}

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









https://lookerstudio.google.com/u/0/reporting/512b58d5-1cbb-4203-8319-4e73df4e509f/page/bdZbD


В районе 8 августа 2024 пересели на даталенс.

*Figma*

***Без Андрея Волкова, Владимира Коршунова, Егора Лапина, Александра Гвоздарева и Даниила Орехова этот проект не был бы реализован.***

