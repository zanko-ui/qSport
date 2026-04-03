# qSport
Volleyball RSVP Automation
A lightweight attendance automation system for volleyball trainings using Google Forms, Google Sheets, and Google Apps Script.

Overview

This project automates training attendance for volleyball sessions that happen every Monday and Friday.

The system allows players to:

submit whether they are Coming or Not coming
select their name from a dropdown
submit attendance only for the currently active training date

The system automatically:

opens and closes the Google Form on a schedule
updates the form so that the training date always matches the next active session
stores responses in Google Sheets
displays attendance in a matrix view by player and training date
supports monthly attendance tracking
Features
Google Form

The form contains:

Who is u → dropdown with player names
Training date → auto-filled with the next active training date
Status → Coming / Not coming
Google Apps Script automation

The form is automated to:

Open on Saturday at 20:00 for the upcoming Monday training
Close on Monday at 20:00
Open on Wednesday at 20:00 for the upcoming Friday training
Close on Friday at 20:00

When the form opens:

the Training date question is automatically updated
Saturday opening sets the date to the next Monday
Wednesday opening sets the date to the next Friday
Google Sheets attendance tracking

Responses are stored in Google Sheets with this structure:

Timestamp
Who is u
Training date
Status
Email address

A separate attendance sheet displays:

players as rows
training dates as columns
checkbox-style attendance indicators
totals and payment calculations
Tech Stack
Google Forms
Google Sheets
Google Apps Script
Form Structure

Recommended final form structure:

1. Who is u

Dropdown list of players:

Antea
Antonio
Bruno
Davor
Dino
Filip
Fran
Gaia
Goran
Hrvoje
Igor
Josip
Jozef
Kristóf
Leon
Lucija
Marin
Marko
Naomi
Nicolas
Nikola
Petra
Ren
Sebastijan
Slaven
Tomislav
Vilma
Yasin
2. Training date

Dropdown with one automatically assigned option

3. Status

Multiple choice:

Coming
Not coming
Automation Logic
Opening/closing schedule
Saturday 20:00 → form opens for Monday
Monday 20:00 → form closes
Wednesday 20:00 → form opens for Friday
Friday 20:00 → form closes
Date assignment

When the form opens:

if opened on Saturday, Training date becomes the upcoming Monday
if opened on Wednesday, Training date becomes the upcoming Friday
Apps Script Example
const TIMEZONE = 'Europe/Zagreb';
const DATE_QUESTION_TITLE = 'Training date';

function getBoundForm_() {
  const formUrl = SpreadsheetApp.getActiveSpreadsheet().getFormUrl();
  if (!formUrl) throw new Error('This Sheet is not connected to a Google Form.');
  return FormApp.openByUrl(formUrl);
}

function openForMonday() {
  const form = getBoundForm_();
  const monday = nextTargetWeekday_(1);
  const dateLabel = formatDate_(monday);

  setDateQuestionByTitle_(form, dateLabel);
  form.setAcceptingResponses(true);
}

function closeAfterMonday() {
  const form = getBoundForm_();
  form.setAcceptingResponses(false);
}

function openForFriday() {
  const form = getBoundForm_();
  const friday = nextTargetWeekday_(5);
  const dateLabel = formatDate_(friday);

  setDateQuestionByTitle_(form, dateLabel);
  form.setAcceptingResponses(true);
}

function closeAfterFriday() {
  const form = getBoundForm_();
  form.setAcceptingResponses(false);
}

function setDateQuestionByTitle_(form, value) {
  const items = form.getItems();

  for (const item of items) {
    if (item.getTitle() !== DATE_QUESTION_TITLE) continue;

    if (item.getType() === FormApp.ItemType.LIST) {
      item.asListItem().setChoiceValues([value]);
      return;
    }

    if (item.getType() === FormApp.ItemType.MULTIPLE_CHOICE) {
      item.asMultipleChoiceItem().setChoiceValues([value]);
      return;
    }

    throw new Error('Training date question must be Dropdown or Multiple choice.');
  }

  throw new Error('Training date question not found.');
}

function nextTargetWeekday_(targetWeekday) {
  const now = new Date();
  const jsDay = now.getDay();
  const weekday = jsDay === 0 ? 7 : jsDay;

  let diff = targetWeekday - weekday;
  if (diff < 0) diff += 7;
  if (diff === 0) diff = 7;

  const result = new Date(now);
  result.setDate(now.getDate() + diff);
  result.setHours(0, 0, 0, 0);
  return result;
}

function formatDate_(date) {
  return Utilities.formatDate(date, TIMEZONE, 'dd/MM/yyyy');
}
Spreadsheet Logic
Helper sheet

The helper sheet mirrors the raw form responses and is used as the source for attendance formulas.

Current response structure:

A = Timestamp
B = Who is u
C = Training date
D = Status
E = Email address
Attendance formula

Used to mark whether a player is coming for a given date:

=IFERROR(
  INDEX(
    FILTER(
      Helper!$D:$D;
      Helper!$B:$B=$A3;
      TEXT(Helper!$C:$C;"dd/mm/yyyy")=TEXT(H$1;"dd/mm/yyyy")
    );
    1
  )="Coming";
  FALSE
)

This checks:

player name in column A
training date in row 1
matching form response in the Helper sheet
Conditional formatting example

To highlight rows green if either checkbox in column C or E is checked:

=OR($C3; $E3)
Recommended Google Form settings

Use:

Collect email addresses
Allow response editing
Limit to 1 response

This keeps the data cleaner and avoids duplicate submissions from the same player.

Monthly Tracking

The spreadsheet supports monthly attendance sheets such as:

TRAVANJ
SVIBANJ
LIPANJ

Each monthly sheet can:

show players as rows
show training dates as columns
calculate payment totals
apply conditional formatting
preserve previous months without overwriting them
Use Case

This system is useful for:

recreational volleyball teams
training attendance tracking
payment calculation based on attendance
quick public view of who is coming
replacing manual WhatsApp attendance polls
Future Improvements

Possible upgrades:

public dashboard for current session attendance
automatic “who did not respond” list
automatic payment summary per month
attendance percentage per player
late cancellation tracking
coach/admin dashboard
Notes
Google Apps Script triggers should be configured manually:
openForMonday
closeAfterMonday
openForFriday
closeAfterFriday
The Training date field must remain a Dropdown or Multiple choice question for the script to update it automatically.
If the form structure changes, formulas and script references may need to be updated.
