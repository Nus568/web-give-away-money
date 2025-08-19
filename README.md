# web-give-away-money

function doGet(e) {
  var x = HtmlService.createTemplateFromFile("login");
  var y = x.evaluate();
  var z = y.setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
  return z;
}

// ฟังก์ชันเช็คการเข้าสู่ระบบ
function checkLogin(username, password) {
  var url = 'https://docs.google.com/spreadsheets/d/1q3a4NVGX-EWTHmxUT-gi9Seyk6AwDJmDkw2jad1xFrY/edit?gid=0#gid=0';
  var ss = SpreadsheetApp.openByUrl(url);
  var sheet = ss.getSheetByName("DATA");
  var data = sheet.getDataRange().getValues(); // ดึงข้อมูลทั้งหมดในคราวเดียว
  var foundUser = 'FALSE';

  for (var i = 1; i < data.length; i++) { // เริ่มที่ index 1 (ข้าม header)
    var sheetUsername = data[i][0].toString().trim();
    var sheetPassword = data[i][1].toString().trim();
    var sheetRole = data[i][10].toString().trim();

    if (sheetUsername === username && sheetPassword === password) {
      foundUser = (sheetRole === 'admin') ? 'admin' : 'user';
      break;
    }
  }
  return foundUser;
}

// ฟังก์ชันเพิ่มข้อมูลใหม่
function AddRecord(username, password, title, firstname, lastname, age, job, income, address, phone) {
  var url = 'https://docs.google.com/spreadsheets/d/1q3a4NVGX-EWTHmxUT-gi9Seyk6AwDJmDkw2jad1xFrY/edit?gid=0#gid=0';
  var ss = SpreadsheetApp.openByUrl(url);
  var sheet = ss.getSheetByName("DATA");

  if (checkUsernameExists(username, sheet)) {
    return 'Username already exists'; // ป้องกันชื่อซ้ำ
  }

  var hashedPassword = hashPassword(password); // เข้ารหัสรหัสผ่านก่อนเก็บ

  sheet.appendRow([username, hashedPassword, title, firstname, lastname, age, job, income, address, phone, 'user']);
  return 'Account created successfully';
}

// ฟังก์ชันตรวจสอบว่ามี username ซ้ำหรือไม่
function checkUsernameExists(username, sheet) {
  var data = sheet.getDataRange().getValues();
  for (var i = 1; i < data.length; i++) {
    if (data[i][0].toString().trim() === username) {
      return true;
    }
  }
  return false;
}

// ฟังก์ชันเข้ารหัสรหัสผ่าน (ใช้ SHA-256)ทำให้แสดงเป็นรัหสเข้ารัหส
function hashPassword(password) {
  return Utilities.computeDigest(Utilities.DigestAlgorithm.SHA_256, password)
    .map(function(b) { return ('0' + (b & 0xFF).toString(16)).slice(-2); })
    .join('');
}
function getAllUsers() {
  var url = 'https://docs.google.com/spreadsheets/d/1q3a4NVGX-EWTHmxUT-gi9Seyk6AwDJmDkw2jad1xFrY/edit?gid=0#gid=0';
  var ss = SpreadsheetApp.openByUrl(url);
  var sheet = ss.getSheetByName("DATA");
  var data = sheet.getDataRange().getValues();

  var users = [];
  for (var i = 1; i < data.length; i++) {
    users.push({
      username: data[i][0],   // Username
      password: data[i][1],   // Password
      title: data[i][2],      // คำนำหน้า
      firstname: data[i][3],  // ชื่อ
      lastname: data[i][4],   // นามสกุล
      age: data[i][5],        // อายุ
      job: data[i][6],        // อาชีพ
      income: data[i][7],     // รายได้
      address: data[i][8],    // ที่อยู่
      phone: data[i][9],      // เบอร์โทร
      role: data[i][10],       // บทบาท (Admin/User)
      number: data[i][11],       // บทบาท (Admin/User)
      allusere: data[i][13],       // บทบาท (Admin/User)
      show: data[i][15],       // บทบาท (Admin/User)
      
    });
  }
  return users;
}
function deleteUserFromSheet(username) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("DATA"); 
  var data = sheet.getDataRange().getValues();
  
  for (var i = 1; i < data.length; i++) { // เริ่มจาก index 1 เพื่อข้ามหัวตาราง
    if (data[i][0] === username) { // สมมติว่า username อยู่ในคอลัมน์แรก
      sheet.deleteRow(i + 1); // ลบแถว (บวก 1 เพราะ index เริ่มที่ 1)
      return "User deleted";
    }
  }
  return "User not found";
}
function addUserFromAdmin(username, password, title, firstname, lastname, age, job, income, address, phone, role) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("DATA");
  var data = sheet.getDataRange().getValues();
  
  // ตรวจสอบว่ามี username ซ้ำหรือไม่
  for (var i = 1; i < data.length; i++) {
    if (data[i][0] === username) {
      return "Username already exists";
    }
  }

  // เพิ่มข้อมูลผู้ใช้ลงใน Google Sheets พร้อมกับบทบาท (role)
  sheet.appendRow([username, password, title, firstname, lastname, age, job, income, address, phone, role]);
  
  return "User added successfully";
}


function addRandomUsers(num) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("DATA");
  if (!sheet) {
    sheet = SpreadsheetApp.getActiveSpreadsheet().insertSheet("DATA");
    sheet.appendRow(["Username", "Password", "Title", "Firstname", "Lastname", "Age", "Job", "Income", "Address", "Phone", "Role"]);
  }

  var titles = ["นาย", "นาง", "นางสาว"];
  var firstNames = ["สมชาย", "สมหญิง", "วราภรณ์", "กิตติพงษ์", "ชลธิชา"];
  var lastNames = ["ใจดี", "บุญมี", "รัตนาวดี", "วงศ์ศรี", "สถาพร"];
  var occupations = ["นักเรียน", "พนักงานบริษัท", "เกษตรกร", "อื่นๆ", "ฟรีแลนซ์","เจ้าของธุรกิจ"];
  
  var data = [];
  for (var i = 0; i < num; i++) {
    var username = "user" + Math.floor(10000 + Math.random() * 90000); 
    var password = hashPassword("password123"); // ใช้รหัสผ่านเริ่มต้นแล้วเข้ารหัส
    var title = titles[Math.floor(Math.random() * titles.length)];
    var firstname = firstNames[Math.floor(Math.random() * firstNames.length)];
    var lastname = lastNames[Math.floor(Math.random() * lastNames.length)];
    var age = Math.floor(Math.random() * 60) + 18;
    var job = occupations[Math.floor(Math.random() * occupations.length)];
    var income = Math.floor(Math.random() * 50000) + 5000;
    var incomeList = ["ต่ำกว่า 10,000","มากกว่า 50,000","10,000 - 30,000"];
    var income = incomeList[Math.floor(Math.random() * incomeList.length)];
    var addressList = ["ชัยนาท","ชลบุรี","ฉะเชิงเทรา","จันทบุรี","ขอนแก่น","กำแพงเพชร","กาฬสินธุ์","กาญจนบุรี","กระบี่ "];
    var address = addressList[Math.floor(Math.random() * addressList.length)];
    var phone = "08" + Math.floor(10000000 + Math.random() * 90000000);
    var role = "user"; 

    data.push([username, password, title, firstname, lastname, age, job, income, address, phone, role]);
  }
  //เข้าถึงข้อมูล sheet อ่านและเขียน
  sheet.getRange(sheet.getLastRow() + 1, 1, data.length, data[0].length).setValues(data);
  return num + " users added successfully";
}
var SHEET_ID = "1q3a4NVGX-EWTHmxUT-gi9Seyk6AwDJmDkw2jad1xFrY";
var sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName("DATA");

function parseIncome(incomeStr) {
  if (typeof incomeStr !== "string") return 0;
  if (incomeStr.includes("มากกว่า")) return parseInt(incomeStr.replace(/[^0-9]/g, "")) + 1;
  var match = incomeStr.match(/\d+/g);
  return match ? parseInt(match[0]) : 0;
}

function getFundBalance() {
  return sheet.getRange("O2").getValue();
}

function setFundBalance(amount) {
  sheet.getRange("O2").setValue(amount);
}

function distributeFunds(round, job, maxIncome, perPersonAmount) {
  var totalFund = getFundBalance();
  var eligibleRecipients = [];
  var data = sheet.getDataRange().getValues();

  for (var i = 1; i < data.length; i++) {
    var personJob = data[i][6]; // คอลัมน์อาชีพ
    var income = data[i][7] ? parseIncome(data[i][7]) : 0; // คอลัมน์รายได้
    var status = data[i][17]; // คอลัมน์ status (ตำแหน่งที่ 18)

    if (personJob === job && income <= maxIncome && (status === "" || status === "รอคิว")) {
      eligibleRecipients.push(i);
    }
  }

  if (eligibleRecipients.length === 0) {
    return `ไม่มีผู้รับสิทธิ์สำหรับอาชีพ ${job} และรายได้ไม่เกิน ${maxIncome} บาท`;
  }

  var requiredAmount = eligibleRecipients.length * perPersonAmount;
  if (totalFund < requiredAmount) {
    perPersonAmount = Math.floor(totalFund / eligibleRecipients.length);
  }

  var distributedAmount = 0;
  for (var i = 0; i < eligibleRecipients.length; i++) {
    if (totalFund >= perPersonAmount) {
      var rowIndex = eligibleRecipients[i] + 1;

      sheet.getRange(rowIndex, 16).setValue(perPersonAmount); // คอลัมน์ show mony (ตำแหน่งที่ 16)
      sheet.getRange(rowIndex, 17).setValue(round === 4 ? "ไม่ผ่านเกณฑ์" : "ได้รับเงินแล้ว"); // คอลัมน์ status (ตำแหน่งที่ 19)

      distributedAmount += perPersonAmount;
      totalFund -= perPersonAmount;
    }
  }

  setFundBalance(totalFund);
  return `แจกเงินรอบที่ ${round} ให้ ${eligibleRecipients.length} คน คนละ ${perPersonAmount} บาท สำเร็จ! คงเหลือเงินกองกลาง ${totalFund} บาท`;
}

function getAllStatuses() {
  var data = sheet.getDataRange().getValues();
  var output = "รายชื่อและสถานะ:\n";

  for (var i = 1; i < data.length; i++) {
    output += `${data[i][0]} - ${data[i][16] || "รอคิว"}\n`; // ชื่อ + สถานะ
  }

  return output;
}
function getAllStatuses2() {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("DATA");
    if (!sheet) return "ERROR: Sheet not found";

    var data = sheet.getDataRange().getValues();
    if (data.length < 2) return "ERROR: No data available";

    var result = [];
    for (var i = 1; i < data.length; i++) { 
        result.push([data[i][0], data[i][16]]); // คอลัมน์ A = index 0, คอลัมน์ Q = index 16
    }

    return result;
}

function addFund(amount) {
  if (amount <= 0 || isNaN(amount)) {
    return "จำนวนเงินต้องมากกว่า 0";
  }

  var sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName("DATA");
  var currentBalance = sheet.getRange("O2").getValue();
  var newBalance = currentBalance + amount;

  sheet.getRange("O2").setValue(newBalance);
  
  return `เพิ่มเงิน ${amount} บาทแล้ว! ยอดเงินคงเหลือ: ${newBalance} บาท`;
}
