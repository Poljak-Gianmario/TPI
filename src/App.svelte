<script>
  //Todo: refresh the calendar, save on .json file the data
  import Calendar from "./Calendar.svelte";
  import { interrogazioni } from "./store";
  var items = [];
  var dayNames = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
  let monthNames = [
    "January",
    "February",
    "March",
    "April",
    "May",
    "June",
    "July",
    "August",
    "September",
    "October",
    "November",
    "December",
  ];
  let headers = [];
  let now = new Date();
  let year = now.getFullYear(); //	this is the month & year displayed
  let month = now.getMonth();
  let eventText = "Click an item or date";
  let nuovoTodo, priority, date, anno, mese, giorno;
  var days = [];
  let allitems = [];
  //	The days to display in each box
  //	The Calendar Component just displays stuff in a row & column. It has no knowledge of dates.
  //	The items[] below are placed (by you) in a specified row & column of the calendar.
  //	You need to call findRowCol() to calc the row/col based on each items start date. Each date        box has a Date() property.
  //	And, if an item overlaps rows, then you need to add a 2nd item on the subsequent row.

  function AddTodo() {
    var ele = document.getElementsByName("priority");
    for (let i = 0; i < ele.length; i++) {
      if (ele[i].checked) {
        priority = ele[i].value;
        break;
      }
    }
    //Formattazione dei dati
    anno = parseInt(date.substring(0, 4));
    mese = parseInt(date.substring(6, 8)) - 1;
    giorno = parseInt(date.substring(8, 10));

    interrogazioni.update((oldValue) => {
      oldValue.push({
        titolo: nuovoTodo,
        anno: anno,
        mese: mese,
        giorno: giorno,
        priorita: priority,
      });
      return oldValue;
    });
    allitems = overwrite();
    initContent();
  //console.log(JSON.stringify(items));
  }
  
  function overwrite() {
    var array = [];
    for (var i = 0; i < $interrogazioni.length; i++) {
      array[i] = {
        title: $interrogazioni[i].titolo,
        className: $interrogazioni[i].priorita,
        date: new Date(
          $interrogazioni[i].anno,
          $interrogazioni[i].mese,
          $interrogazioni[i].giorno
        ),
        len: 1,
        id: i,
      };
    }
    return array;
  }
  
  function getMonthItems() {
    var array = [];
    var index = 0;
    for (let i of allitems) {
      if (i.date.getMonth() == month) {
        array[index] = i;
        index++;
      }
    }
    return array;
  }
  
  function initMonthItems() {
    items = getMonthItems();
    //This is where you calc the row/col to put each dated item
    for (let i of items) {
      console.log(i);
      let rc = findRowCol(i.date);
      if (rc == null) {
        console.log("didn`t find date for ", i);
        console.log(i.date);
        console.log(days);
        i.startCol = i.startRow = 0;
      } else {
        i.startCol = rc.col;
        i.startRow = rc.row;
      }
    }
  }
  
  $: month, year, initContent();
  // choose what date/day gets displayed in each date box.
  
  function initContent() {
    headers = dayNames;
    initMonth();
    initMonthItems();
  }

  function initMonth() {
    days = [];
    let monthAbbrev = monthNames[month].slice(0, 3);
    let nextMonthAbbrev = monthNames[(month + 1) % 12].slice(0, 3);
    //	find the last Monday of the previous month
    var firstDay = new Date(year, month, 1).getDay();
    //console.log('fd='+firstDay+' '+dayNames[firstDay]);
    var daysInThisMonth = new Date(year, month + 1, 0).getDate();
    var daysInLastMonth = new Date(year, month, 0).getDate();
    var prevMonth = month == 0 ? 11 : month - 1;

    //	show the days before the start of this month (disabled) - always less than 7
    for (let i = daysInLastMonth - firstDay; i < daysInLastMonth; i++) {
      let d = new Date(prevMonth == 11 ? year - 1 : year, prevMonth, i + 1);
      days.push({ name: "" + (i + 1), enabled: false, date: d });
    }
    //	show the days in this month (enabled) - always 28 - 31
    for (let i = 0; i < daysInThisMonth; i++) {
      let d = new Date(year, month, i + 1);
      if (i == 0)
        days.push({
          name: monthAbbrev + " " + (i + 1),
          enabled: true,
          date: d,
        });
      else days.push({ name: "" + (i + 1), enabled: true, date: d });
      //console.log('i='+i+'  dt is '+d+' date() is '+d.getDate());
    }
    //	show any days to fill up the last row (disabled) - always less than 7
    for (let i = 0; days.length % 7; i++) {
      let d = new Date(month == 11 ? year + 1 : year, (month + 1) % 12, i + 1);
      if (i == 0)
        days.push({
          name: nextMonthAbbrev + " " + (i + 1),
          enabled: false,
          date: d,
        });
      else days.push({ name: "" + (i + 1), enabled: false, date: d });
    }
  }

  function findRowCol(dt) {
    for (let i = 0; i < days.length; i++) {
      let d = days[i].date;
      if (
        d.getYear() === dt.getYear() &&
        d.getMonth() === dt.getMonth() &&
        d.getDate() === dt.getDate()
      )
        return { row: Math.floor(i / 7) + 2, col: (i % 7) + 1 };
    }
    return null;
  }

  function itemClick(e) {
    eventText =
      "Task: " + e.title;
  }
  
  function headerClick(e) {
    eventText = "onHheaderClick " + JSON.stringify(e);
  }
  
  function next() {
    month++;
    if (month == 12) {
      year++;
      month = 0;
    }
    initContent();
  }
  
  function prev() {
    if (month == 0) {
      month = 11;
      year--;
    } else {
      month--;
    }
    initContent();
  }
  
  function refresh(event) {
    console.log("Updated the calendar");
  }
  
</script>

<div class="calendar-container">
  <div class="calendar-header">
    <h1>
      <button on:click={() => year--}>&Lt;</button>
      <button on:click={() => prev()}>&lt;</button>
      {monthNames[month]}
      {year}
      <button on:click={() => next()}>&gt;</button>
      <button on:click={() => year++}>&Gt;</button>
    </h1>
    {eventText}
  </div>

  <Calendar
    {headers}
    {days}
    {items}
    on:scroll={refresh()}
    on:itemClick={(e) => itemClick(e.detail)}
    on:headerClick={(e) => headerClick(e.detail)}
  />
</div>
<input type="checkbox" id="my-modal" class="modal-toggle" />
    <div class="modal">
      <div class="modal-box">
        <h3 class="font-bold text-lg">Aggiungi un evento al calendario</h3>
        <br>
        <input type="text" bind:value={nuovoTodo} placeholder="Titolo"/>
        <br><br>  
        <input type="date" bind:value={date}/>
        <br><br>
        <div class=c>
        <input type="radio"  name="priority" value="task--danger"/>Alta 
        <input type="radio"  name="priority" value="task-warning"/>Media
        <input type="radio"  value ="task--info" name="priority"/>Bassa
        </div> 
        <div class="modal-action">
          <label for="my-modal" class="btn" on:click={AddTodo}>Ok</label>
        </div>
      </div>
    </div>

<input type="checkbox" id="my-modal" class="modal-toggle" />
    <div class="modal">
      <div class="modal-box">
        <h3 class="font-bold text-lg">Rim un evento al calendario</h3>
        <input type="text" bind:value={nuovoTodo} placeholder="Titolo"/>
        <br>  
        <input type="date" bind:value={date}/>
        <br>
        <input type="radio"  name="priority" value="task--danger"/>Alta
        <input type="radio"  name="priority" value="task-warning"/>Media
        <input type="radio"  value ="task--info" name="priority"/>Bassa
        <br>  
        <div class="modal-action">
          <label for="my-modal" class="btn" on:click={AddTodo}>Ok</label>
        </div>
      </div>
    </div>

<style>
  .calendar-container {
    width: 90%;
    margin: auto;
    overflow: hidden;
    box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
    border-radius: 10px;
    background: #fff;
    max-width: 1200px;
  }
  .calendar-header {
    text-align: center;
    padding: 20px 0;
    background: #eef;
    border-bottom: 1px solid rgba(166, 168, 179, 0.12);
  }
  .calendar-header h1 {
    margin: 0;
    font-size: 18px;
  }
  .calendar-header button {
    background: #eef;
    border: 1px;
    padding: 6;
    color: rgba(81, 86, 93, 0.7);
    cursor: pointer;
    outline: 0;
  }
</style>
