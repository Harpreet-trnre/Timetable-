<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Student Timetable with Reminders</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    table { border-collapse: collapse; width: 100%; margin-top: 20px; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
    th { background: #f0f0f0; }
    #event-form { margin-bottom: 20px; }
    #event-form input, #event-form select { margin-right: 8px; }
    .reminder-set { color: green; font-weight: bold; }
  </style>
</head>
<body>
  <h2>Student Timetable with Reminders</h2>

  <form id="event-form">
    <input type="text" id="subject" placeholder="Subject" required>
    <select id="day">
      <option value="Monday">Monday</option>
      <option value="Tuesday">Tuesday</option>
      <option value="Wednesday">Wednesday</option>
      <option value="Thursday">Thursday</option>
      <option value="Friday">Friday</option>
      <option value="Saturday">Saturday</option>
      <option value="Sunday">Sunday</option>
    </select>
    <input type="time" id="time" required>
    <input type="number" id="reminder" placeholder="Reminder (mins before)" min="1">
    <button type="submit">Add Event</button>
  </form>

  <table id="timetable">
    <thead>
      <tr>
        <th>Day</th>
        <th>Time</th>
        <th>Subject</th>
        <th>Reminder</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <script>
    // Request notification permission
    if ('Notification' in window && Notification.permission !== 'granted') {
      Notification.requestPermission();
    }

    const events = [];

    document.getElementById('event-form').addEventListener('submit', function(e) {
      e.preventDefault();
      const subject = document.getElementById('subject').value;
      const day = document.getElementById('day').value;
      const time = document.getElementById('time').value;
      const reminder = parseInt(document.getElementById('reminder').value, 10);

      events.push({ subject, day, time, reminder });
      renderTable();
      setReminder(events.length - 1);
      this.reset();
    });

    function renderTable() {
      const tbody = document.querySelector('#timetable tbody');
      tbody.innerHTML = '';
      events.forEach((event, idx) => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${event.day}</td>
          <td>${event.time}</td>
          <td>${event.subject}</td>
          <td>${event.reminder ? `<span class="reminder-set">${event.reminder} min before</span>` : 'No reminder'}</td>
        `;
        tbody.appendChild(row);
      });
    }

    function setReminder(idx) {
      const event = events[idx];
      if (!event.reminder) return;

      // Calculate time till event
      const now = new Date();
      const eventDayNum = ["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"].indexOf(event.day);
      let eventDate = new Date(now);
      eventDate.setDate(now.getDate() + ((eventDayNum + 7 - now.getDay()) % 7));
      const [h, m] = event.time.split(':');
      eventDate.setHours(h, m, 0, 0);

      const reminderTime = new Date(eventDate.getTime() - event.reminder * 60000);

      const msUntilReminder = reminderTime - now;
      if (msUntilReminder > 0) {
        setTimeout(() => {
          if (Notification.permission === "granted") {
            new Notification(`Reminder: ${event.subject} at ${event.time} on ${event.day}`);
          } else {
            alert(`Reminder: ${event.subject} at ${event.time} on ${event.day}`);
          }
        }, msUntilReminder);
      }
    }
  </script>
</body>
</html>
