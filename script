// ============================================================
// SEMESTER COUNTER
// Scriptable Widget
//
// Datenquelle:
// https://lukasedelmann.com/semester.json
//
// Größen:
//   small
//   medium
//   large
//
// Optional für Semesterfortschritt:
//   "semesterStart": "2026-09-14"
//   "semesterEnd":   "2027-02-28"
// ============================================================


// ------------------------------------------------------------
// KONFIGURATION
// ------------------------------------------------------------

const DATA_URL =
  "https://lukasedelmann.com/semester.json";

const REFRESH_MINUTES = 30;

const MAX_LATER_ITEMS = 5;


// ------------------------------------------------------------
// FARBEN
// ------------------------------------------------------------

const C = {

  background: Color.dynamic(
    new Color("#F5F5F7"),
    new Color("#161617")
  ),

  primary: Color.dynamic(
    new Color("#111111"),
    new Color("#F5F5F7")
  ),

  secondary: Color.dynamic(
    new Color("#747479"),
    new Color("#9B9BA1")
  ),

  separator: Color.dynamic(
    new Color("#D7D7DC"),
    new Color("#343438")
  ),

  accent: Color.dynamic(
    new Color("#0066CC"),
    new Color("#4DA3FF")
  ),

  highlight: Color.dynamic(
    new Color("#007AFF", 0.10),
    new Color("#0A84FF", 0.16)
  ),

  progressTrack: Color.dynamic(
    new Color("#D7D7DC"),
    new Color("#343438")
  ),

  error: Color.dynamic(
    new Color("#B42318"),
    new Color("#FF6961")
  )
};


// ------------------------------------------------------------
// DATEN LADEN
// ------------------------------------------------------------

async function loadData() {

  const request =
    new Request(DATA_URL);

  request.timeoutInterval = 10;

  const data =
    await request.loadJSON();


  if (
    !data ||
    !Array.isArray(data.items)
  ) {

    throw new Error(
      "Ungültige semester.json"
    );
  }


  return data;
}


// ------------------------------------------------------------
// DATUM
// ------------------------------------------------------------

function parseDate(dateString) {

  if (!dateString) {
    return null;
  }


  const parts =
    dateString
      .split("-")
      .map(Number);


  if (parts.length !== 3) {
    return null;
  }


  const [year, month, day] =
    parts;


  return new Date(
    year,
    month - 1,
    day,
    12,
    0,
    0
  );
}


function todayDate() {

  const now =
    new Date();


  return new Date(
    now.getFullYear(),
    now.getMonth(),
    now.getDate(),
    12,
    0,
    0
  );
}


function calendarSerial(date) {

  return Math.floor(
    Date.UTC(
      date.getFullYear(),
      date.getMonth(),
      date.getDate()
    ) / 86400000
  );
}


function calendarDaysBetween(
  from,
  to
) {

  return (
    calendarSerial(to) -
    calendarSerial(from)
  );
}


// ------------------------------------------------------------
// EVENT MIT UHRZEIT
// ------------------------------------------------------------

function eventDateTime(item) {

  const date =
    parseDate(item.start);


  if (!date) {
    return null;
  }


  if (!item.time) {
    return date;
  }


  const parts =
    item.time
      .split(":")
      .map(Number);


  if (parts.length !== 2) {
    return date;
  }


  date.setHours(
    parts[0],
    parts[1],
    0,
    0
  );


  return date;
}


// ------------------------------------------------------------
// ITEM STATUS
// ------------------------------------------------------------

function isActive(item) {

  const today =
    todayDate();


  // ZEITRAUM
  if (item.type === "period") {

    const start =
      parseDate(item.start);

    const end =
      parseDate(item.end);


    if (!start || !end) {
      return false;
    }


    return (
      calendarSerial(today) >=
        calendarSerial(start) &&
      calendarSerial(today) <=
        calendarSerial(end)
    );
  }


  // EINZELTERMIN
  if (item.type === "event") {

    const date =
      parseDate(item.start);


    if (!date) {
      return false;
    }


    const sameDay =
      calendarSerial(date) ===
      calendarSerial(today);


    if (!sameDay) {
      return false;
    }


    // Ohne Uhrzeit ganztägig aktiv
    if (!item.time) {
      return true;
    }


    return (
      eventDateTime(item) >=
      new Date()
    );
  }


  return false;
}


function isFuture(item) {

  const today =
    todayDate();

  const start =
    parseDate(item.start);


  if (!start) {
    return false;
  }


  const difference =
    calendarSerial(start) -
    calendarSerial(today);


  if (difference > 0) {
    return true;
  }


  // heutiger Event mit späterer Uhrzeit
  if (
    difference === 0 &&
    item.type === "event" &&
    item.time
  ) {

    return (
      eventDateTime(item) >
      new Date()
    );
  }


  return false;
}


// ------------------------------------------------------------
// PRIORITÄT & SORTIERUNG
// ------------------------------------------------------------

function priority(item) {

  const p =
    Number(item.priority);


  return Number.isFinite(p)
    ? p
    : 0;
}


function itemStartTime(item) {

  if (
    item.type === "event" &&
    item.time
  ) {

    return (
      eventDateTime(item)?.getTime() ??
      Infinity
    );
  }


  return (
    parseDate(item.start)?.getTime() ??
    Infinity
  );
}


function getActiveItems(items) {

  return items
    .filter(isActive)
    .sort((a, b) => {

      const priorityDifference =
        priority(b) -
        priority(a);


      if (priorityDifference !== 0) {
        return priorityDifference;
      }


      // Einzeltermin gewinnt bei gleicher Priorität
      if (a.type !== b.type) {

        return (
          a.type === "event"
            ? -1
            : 1
        );
      }


      return (
        itemStartTime(a) -
        itemStartTime(b)
      );
    });
}


function getFutureItems(items) {

  return items
    .filter(isFuture)
    .sort((a, b) => {

      const timeDifference =
        itemStartTime(a) -
        itemStartTime(b);


      if (timeDifference !== 0) {
        return timeDifference;
      }


      return (
        priority(b) -
        priority(a)
      );
    });
}


// ------------------------------------------------------------
// TEXTFORMATIERUNG
// ------------------------------------------------------------

const MONTHS = [
  "Jan",
  "Feb",
  "Mär",
  "Apr",
  "Mai",
  "Jun",
  "Jul",
  "Aug",
  "Sep",
  "Okt",
  "Nov",
  "Dez"
];


function formatDate(dateString) {

  const date =
    parseDate(dateString);


  if (!date) {
    return "—";
  }


  return (
    `${String(date.getDate()).padStart(2, "0")}. ` +
    MONTHS[date.getMonth()]
  );
}


function formatDateRange(item) {

  if (item.type === "event") {

    let result =
      formatDate(item.start);


    if (item.time) {

      result +=
        ` · ${item.time}`;
    }


    return result;
  }


  const start =
    formatDate(item.start);

  const end =
    formatDate(item.end);


  if (start === end) {
    return start;
  }


  return `${start} – ${end}`;
}


function formatToday() {

  const now =
    new Date();


  return (
    `${String(now.getDate()).padStart(2, "0")}. ` +
    MONTHS[now.getMonth()].toUpperCase()
  );
}


// ------------------------------------------------------------
// COUNTER
// ------------------------------------------------------------

function daysUntil(item) {

  const start =
    parseDate(item.start);


  if (!start) {
    return null;
  }


  return calendarDaysBetween(
    todayDate(),
    start
  );
}


function daysRemaining(item) {

  if (item.type !== "period") {
    return 0;
  }


  const end =
    parseDate(item.end);


  if (!end) {
    return null;
  }


  return calendarDaysBetween(
    todayDate(),
    end
  );
}


function activeCounter(item) {

  if (item.type === "event") {
    return "Heute";
  }


  const days =
    daysRemaining(item);


  if (days == null) {
    return "";
  }


  if (days <= 0) {
    return "Letzter Tag";
  }


  if (days === 1) {
    return "Noch 1 Tag";
  }


  return `Noch ${days} Tage`;
}


function futureCounter(item) {

  const days =
    daysUntil(item);


  if (days == null) {
    return "";
  }


  if (days <= 0) {
    return "Heute";
  }


  if (days === 1) {
    return "Morgen";
  }


  return `In ${days} Tagen`;
}


function shortFutureCounter(item) {

  const days =
    daysUntil(item);


  if (days == null) {
    return "";
  }


  if (days <= 0) {
    return "heute";
  }


  if (days === 1) {
    return "morgen";
  }


  return `in ${days} T.`;
}


function activeDetail(item) {

  if (item.type === "event") {
    return formatDateRange(item);
  }


  return `bis ${formatDate(item.end)}`;
}


// ------------------------------------------------------------
// TEXT-HELPER
// ------------------------------------------------------------

function addText(
  stack,
  value,
  options = {}
) {

  const text =
    stack.addText(
      String(value ?? "")
    );


  text.font =
    options.font ??
    Font.systemFont(
      options.size ?? 10
    );


  text.textColor =
    options.color ??
    C.primary;


  if (
    options.lineLimit !== undefined
  ) {

    text.lineLimit =
      options.lineLimit;
  }


  if (
    options.minimumScaleFactor !==
    undefined
  ) {

    text.minimumScaleFactor =
      options.minimumScaleFactor;
  }


  return text;
}


function addSectionTitle(
  widget,
  title
) {

  const row =
    widget.addStack();


  const text =
    row.addText(
      title.toUpperCase()
    );


  text.font =
    Font.semiboldSystemFont(8);


  text.textColor =
    C.secondary;
}


function addSeparator(widget) {

  const separator =
    widget.addStack();


  separator.size =
    new Size(0, 0.5);


  separator.backgroundColor =
    C.separator;
}


// ------------------------------------------------------------
// HEADER
// ------------------------------------------------------------

function addHeader(
  widget,
  compact = false
) {

  const header =
    widget.addStack();


  header.centerAlignContent();


  addText(
    header,
    "SEMESTER",
    {
      font:
        Font.boldSystemFont(
          compact
            ? 12
            : 15
        )
    }
  );


  header.addSpacer();


  addText(
    header,
    formatToday(),
    {
      font:
        Font.regularMonospacedSystemFont(
          compact
            ? 7
            : 8
        ),

      color:
        C.secondary
    }
  );
}


// ============================================================
// HIGHLIGHT CARD
// ============================================================

function addHighlightCard(
  widget,
  item,
  state,
  family
) {

  /*
   * Horizontaler äußerer Stack.
   *
   * Der Spacer am Ende sorgt dafür,
   * dass die blaue Fläche die gesamte
   * verfügbare Widgetbreite einnimmt.
   */
  const card =
    widget.addStack();


  card.backgroundColor =
    C.highlight;


  card.cornerRadius =
    family === "small"
      ? 8
      : 7;


  if (family === "small") {

    card.setPadding(
      8,
      8,
      8,
      8
    );

  } else {

    card.setPadding(
      8,
      10,
      8,
      10
    );
  }


  // eigentlicher Inhalt
  const content =
    card.addStack();


  content.layoutVertically();


  // TITEL
  addText(
    content,
    item.title,
    {
      font:
        Font.semiboldSystemFont(
          family === "large"
            ? 14
            : family === "medium"
              ? 12
              : 11
        ),

      lineLimit:
        family === "small"
          ? 2
          : 1,

      minimumScaleFactor:
        0.75
    }
  );


  content.addSpacer(
    family === "large"
      ? 5
      : 3
  );


  // COUNTER
  const counter =
    state === "active"
      ? activeCounter(item)
      : futureCounter(item);


  addText(
    content,
    counter,
    {
      font:
        Font.boldSystemFont(
          family === "small"
            ? 22
            : family === "medium"
              ? 20
              : 25
        ),

      color:
        C.accent,

      minimumScaleFactor:
        0.65,

      lineLimit: 1
    }
  );


  content.addSpacer(2);


  // DATUM
  addText(
    content,
    state === "active"
      ? activeDetail(item)
      : formatDateRange(item),
    {
      font:
        Font.systemFont(
          family === "large"
            ? 10
            : 8
        ),

      color:
        C.secondary,

      lineLimit: 1,

      minimumScaleFactor:
        0.7
    }
  );


  // ----------------------------------------------------------
  // FORTSCHRITT DES AKTUELLEN ZEITRAUMS
  // nur Large
  // ----------------------------------------------------------

  if (
    family === "large" &&
    state === "active" &&
    item.type === "period"
  ) {

    const start =
      parseDate(item.start);

    const end =
      parseDate(item.end);

    const today =
      todayDate();


    const totalDays =
      calendarDaysBetween(
        start,
        end
      ) + 1;


    const currentDay =
      calendarDaysBetween(
        start,
        today
      ) + 1;


    content.addSpacer(3);


    addText(
      content,
      `Tag ${currentDay} von ${totalDays}`,
      {
        font:
          Font.systemFont(8),

        color:
          C.secondary
      }
    );
  }


  /*
   * Entscheidend:
   * füllt den restlichen horizontalen Raum
   * und zieht damit den Hintergrund auf
   * volle Breite.
   */
  card.addSpacer();
}


// ------------------------------------------------------------
// NÄCHSTER EINTRAG
// ------------------------------------------------------------

function addCompactItem(
  widget,
  item
) {

  const row =
    widget.addStack();


  row.centerAlignContent();


  const left =
    row.addStack();


  left.layoutVertically();


  addText(
    left,
    item.title,
    {
      font:
        Font.semiboldSystemFont(10),

      lineLimit: 1,

      minimumScaleFactor: 0.7
    }
  );


  left.addSpacer(1);


  addText(
    left,
    formatDateRange(item),
    {
      font:
        Font.systemFont(8),

      color:
        C.secondary,

      lineLimit: 1
    }
  );


  row.addSpacer();


  addText(
    row,
    futureCounter(item),
    {
      font:
        Font.semiboldSystemFont(10),

      color:
        C.primary,

      lineLimit: 1,

      minimumScaleFactor: 0.7
    }
  );
}


// ------------------------------------------------------------
// SPÄTER-ZEILE
// ------------------------------------------------------------

function addLaterRow(
  widget,
  item
) {

  const row =
    widget.addStack();


  row.centerAlignContent();


  addText(
    row,
    formatDate(item.start),
    {
      font:
        Font.regularMonospacedSystemFont(8),

      color:
        C.secondary
    }
  );


  row.addSpacer(8);


  addText(
    row,
    item.title,
    {
      font:
        Font.systemFont(9),

      lineLimit: 1,

      minimumScaleFactor: 0.7
    }
  );


  row.addSpacer();


  addText(
    row,
    shortFutureCounter(item),
    {
      font:
        Font.semiboldSystemFont(8),

      color:
        C.secondary
    }
  );
}


// ============================================================
// SEMESTERFORTSCHRITT
// ============================================================

function semesterProgress(data) {

  const start =
    parseDate(
      data.semesterStart
    );


  const end =
    parseDate(
      data.semesterEnd
    );


  if (!start || !end) {
    return null;
  }


  const today =
    todayDate();


  const total =
    calendarDaysBetween(
      start,
      end
    );


  if (total <= 0) {
    return null;
  }


  const elapsed =
    calendarDaysBetween(
      start,
      today
    );


  let progress =
    elapsed / total;


  progress =
    Math.max(
      0,
      Math.min(
        1,
        progress
      )
    );


  return {
    progress,
    percent:
      Math.round(
        progress * 100
      )
  };
}


function addSemesterProgress(
  widget,
  data
) {

  const progress =
    semesterProgress(data);


  if (!progress) {
    return;
  }


  addSeparator(widget);

  widget.addSpacer(6);


  const header =
    widget.addStack();


  addText(
    header,
    "SEMESTERFORTSCHRITT",
    {
      font:
        Font.semiboldSystemFont(8),

      color:
        C.secondary
    }
  );


  header.addSpacer();


  addText(
    header,
    `${progress.percent} %`,
    {
      font:
        Font.semiboldSystemFont(8),

      color:
        C.secondary
    }
  );


  widget.addSpacer(4);


  /*
   * Fortschrittsbalken
   */
  const track =
    widget.addStack();


  track.size =
    new Size(0, 5);


  track.backgroundColor =
    C.progressTrack;


  track.cornerRadius = 3;


  /*
   * Scriptable benötigt eine konkrete
   * Breite für den gefüllten Anteil.
   *
   * Large Widgets liegen ungefähr bei
   * 330 logischen Punkten nutzbarer Breite.
   */
  const usableWidth = 300;


  const fill =
    track.addStack();


  fill.size =
    new Size(
      usableWidth *
      progress.progress,
      5
    );


  fill.backgroundColor =
    C.accent;


  fill.cornerRadius = 3;


  track.addSpacer();
}


// ============================================================
// SMALL
// ============================================================

function createSmallWidget(
  data,
  active,
  future
) {

  const widget =
    new ListWidget();


  widget.backgroundColor =
    C.background;


  widget.setPadding(
    10,
    10,
    9,
    10
  );


  addHeader(
    widget,
    true
  );


  widget.addSpacer(7);


  let item;
  let state;


  if (active.length > 0) {

    item =
      active[0];

    state =
      "active";


    addSectionTitle(
      widget,
      item.type === "event"
        ? "HEUTE"
        : "JETZT"
    );

  } else if (future.length > 0) {

    item =
      future[0];

    state =
      "future";


    addSectionTitle(
      widget,
      "ALS NÄCHSTES"
    );

  } else {

    addText(
      widget,
      "Keine kommenden Termine",
      {
        size: 10,
        color: C.secondary
      }
    );


    return widget;
  }


  widget.addSpacer(3);


  addHighlightCard(
    widget,
    item,
    state,
    "small"
  );


  widget.addSpacer();


  addText(
    widget,
    data.semester ?? "",
    {
      font:
        Font.systemFont(7),

      color:
        C.secondary
    }
  );


  return widget;
}


// ============================================================
// MEDIUM
// ============================================================

function createMediumWidget(
  data,
  active,
  future
) {

  const widget =
    new ListWidget();


  widget.backgroundColor =
    C.background;


  widget.setPadding(
    10,
    12,
    8,
    12
  );


  addHeader(
    widget
  );


  widget.addSpacer(5);


  // ----------------------------------------------------------
  // AKTUELLER EINTRAG
  // ----------------------------------------------------------

  if (active.length > 0) {

    addSectionTitle(
      widget,
      active[0].type === "event"
        ? "HEUTE"
        : "JETZT"
    );


    widget.addSpacer(2);


    addHighlightCard(
      widget,
      active[0],
      "active",
      "medium"
    );


    if (future.length > 0) {

      widget.addSpacer(4);

      addSeparator(widget);

      widget.addSpacer(4);


      addSectionTitle(
        widget,
        "ALS NÄCHSTES"
      );


      widget.addSpacer(2);


      addCompactItem(
        widget,
        future[0]
      );
    }
  }


  // ----------------------------------------------------------
  // KEIN AKTUELLER EINTRAG
  // ----------------------------------------------------------

  else if (future.length > 0) {

    addSectionTitle(
      widget,
      "ALS NÄCHSTES"
    );


    widget.addSpacer(2);


    addHighlightCard(
      widget,
      future[0],
      "future",
      "medium"
    );


    if (future.length > 1) {

      widget.addSpacer(4);

      addSeparator(widget);

      widget.addSpacer(4);


      addSectionTitle(
        widget,
        "DANACH"
      );


      widget.addSpacer(2);


      addCompactItem(
        widget,
        future[1]
      );
    }
  }


  else {

    addText(
      widget,
      "Keine kommenden Termine",
      {
        size: 10,
        color: C.secondary
      }
    );
  }


  return widget;
}


// ============================================================
// LARGE
// ============================================================

function createLargeWidget(
  data,
  active,
  future
) {

  const widget =
    new ListWidget();


  widget.backgroundColor =
    C.background;


  widget.setPadding(
    11,
    12,
    9,
    12
  );


  addHeader(
    widget
  );


  widget.addSpacer(7);


  let laterStartIndex = 0;


  // ----------------------------------------------------------
  // AKTUELL
  // ----------------------------------------------------------

  if (active.length > 0) {

    addSectionTitle(
      widget,
      active[0].type === "event"
        ? "HEUTE"
        : "JETZT"
    );


    widget.addSpacer(3);


    addHighlightCard(
      widget,
      active[0],
      "active",
      "large"
    );


    widget.addSpacer(6);

    addSeparator(widget);

    widget.addSpacer(6);


    // --------------------------------------------------------
    // ALS NÄCHSTES
    // --------------------------------------------------------

    if (future.length > 0) {

      addSectionTitle(
        widget,
        "ALS NÄCHSTES"
      );


      widget.addSpacer(3);


      addCompactItem(
        widget,
        future[0]
      );


      laterStartIndex = 1;
    }
  }


  // ----------------------------------------------------------
  // KEIN AKTUELLER ZEITRAUM
  // ----------------------------------------------------------

  else if (future.length > 0) {

    addSectionTitle(
      widget,
      "ALS NÄCHSTES"
    );


    widget.addSpacer(3);


    addHighlightCard(
      widget,
      future[0],
      "future",
      "large"
    );


    laterStartIndex = 1;
  }


  // ----------------------------------------------------------
  // SPÄTER
  // ----------------------------------------------------------

  const later =
    future.slice(
      laterStartIndex,
      laterStartIndex +
        MAX_LATER_ITEMS
    );


  if (later.length > 0) {

    widget.addSpacer(6);

    addSeparator(widget);

    widget.addSpacer(6);


    addSectionTitle(
      widget,
      "SPÄTER"
    );


    widget.addSpacer(3);


    for (
      let i = 0;
      i < later.length;
      i++
    ) {

      addLaterRow(
        widget,
        later[i]
      );


      if (
        i <
        later.length - 1
      ) {

        widget.addSpacer(4);
      }
    }
  }


  /*
   * Restlichen freien Platz nach unten schieben,
   * damit der Semesterfortschritt am unteren Rand
   * sitzt.
   */
  widget.addSpacer();


  // ----------------------------------------------------------
  // SEMESTERFORTSCHRITT
  // ----------------------------------------------------------

  addSemesterProgress(
    widget,
    data
  );


  if (
    !data.semesterStart ||
    !data.semesterEnd
  ) {

    addText(
      widget,
      data.semester ?? "",
      {
        font:
          Font.systemFont(7),

        color:
          C.secondary
      }
    );
  }


  return widget;
}


// ============================================================
// ERROR
// ============================================================

function createErrorWidget(error) {

  const widget =
    new ListWidget();


  widget.backgroundColor =
    C.background;


  widget.setPadding(
    12,
    12,
    12,
    12
  );


  addText(
    widget,
    "SEMESTER",
    {
      font:
        Font.boldSystemFont(14)
    }
  );


  widget.addSpacer(8);


  addText(
    widget,
    "Daten konnten nicht geladen werden.",
    {
      font:
        Font.semiboldSystemFont(10),

      color:
        C.error
    }
  );


  widget.addSpacer(4);


  addText(
    widget,
    String(
      error?.message ??
      error
    ),
    {
      font:
        Font.systemFont(8),

      color:
        C.secondary,

      lineLimit: 3
    }
  );


  return widget;
}


// ============================================================
// WIDGET ERZEUGEN
// ============================================================

async function createWidget(
  family
) {

  try {

    const data =
      await loadData();


    const active =
      getActiveItems(
        data.items
      );


    const future =
      getFutureItems(
        data.items
      );


    let widget;


    switch (family) {

      case "small":

        widget =
          createSmallWidget(
            data,
            active,
            future
          );

        break;


      case "medium":

        widget =
          createMediumWidget(
            data,
            active,
            future
          );

        break;


      case "large":

      default:

        widget =
          createLargeWidget(
            data,
            active,
            future
          );

        break;
    }


    widget.refreshAfterDate =
      new Date(
        Date.now() +
        REFRESH_MINUTES *
        60 *
        1000
      );


    return widget;

  } catch (error) {

    console.error(error);


    return createErrorWidget(
      error
    );
  }
}


// ============================================================
// START
// ============================================================

let family;


// ------------------------------------------------------------
// HOMESCREEN
// ------------------------------------------------------------

if (config.runsInWidget) {

  family =
    config.widgetFamily;

}


// ------------------------------------------------------------
// SCRIPTABLE VORSCHAU
// ------------------------------------------------------------

else {

  const alert =
    new Alert();


  alert.title =
    "Widget-Vorschau";


  alert.addAction(
    "Small"
  );


  alert.addAction(
    "Medium"
  );


  alert.addAction(
    "Large"
  );


  const selection =
    await alert.presentSheet();


  family =
    [
      "small",
      "medium",
      "large"
    ][selection] ??
    "large";
}


// ------------------------------------------------------------
// WIDGET BAUEN
// ------------------------------------------------------------

const widget =
  await createWidget(
    family
  );


// ------------------------------------------------------------
// ANZEIGEN
// ------------------------------------------------------------

if (config.runsInWidget) {

  Script.setWidget(
    widget
  );

} else {

  if (family === "small") {

    await widget.presentSmall();

  }

  else if (
    family === "medium"
  ) {

    await widget.presentMedium();

  }

  else {

    await widget.presentLarge();
  }
}


Script.complete();
