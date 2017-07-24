//対象とするGoogleDriveフォルダのID　ブラウザでアクセスしてURL見れば分かる
var TARGET_FOLDER_ID = "xxxxxxxxxxxxxxxxxxxxxx";
//更新日時を記録するスプレッドシートのID　ブラウザでアクセスしてURL見れば分かる
var UPDATE_SHEET_ID = "xxxxxxxxxxxxxxxxxxxxxx";
//スプレッドシートのシート名（下に表示されるタブのやつ）
var UPDATE_SHEET_NAME = "シート1";
// 送信先のメールアドレス　このスクリプトの実行ユーザーは、送信済みトレイに入るので注意
var SEND_MAIL_ADDRESS = ["user1@example.com", "user2@example.com"]

function updateCheck() {
  var targetFolder = DriveApp.getFolderById(TARGET_FOLDER_ID);
  var folders = targetFolder.getFolders();
  var files = targetFolder.getFiles();

  //フォルダ内を再帰的に探索してすべてのファイルIDを配列にして返す
  function getAllFilesId(targetFolder){
    var filesIdList = [];
    
    var files = targetFolder.getFiles();
    while(files.hasNext()){
      filesIdList.push(files.next().getId());
    }
    
    var child_folders = targetFolder.getFolders();
    while(child_folders.hasNext()){
      var child_folder = child_folders.next();
      //Logger.log( 'child_folder :' + child_folder );
      //Logger.log('getAllFilesId(child_folder):'+ getAllFilesId(child_folder));
      filesIdList = filesIdList.concat( getAllFilesId(child_folder) );
    }
    return filesIdList;
  }
  //Logger.log('getAllFilesId(targetFolder):' + getAllFilesId(targetFolder));
  var allFilesId = getAllFilesId(targetFolder);
  var lastUpdateMap = {};
  //Logger.log(folders)
  allFilesId.forEach(
    function( value, i ){
      var file =DriveApp.getFileById( value );
      lastUpdateMap[file.getName()] = {lastUpdate : file.getLastUpdated(), fileId: file.getId()};
    }
  );          
 
  // スプレッドシートに記載されているフォルダ名と更新日時を取得。
  var spreadsheet = SpreadsheetApp.openById(UPDATE_SHEET_ID);
  var sheet = spreadsheet.getSheetByName(UPDATE_SHEET_NAME);
  //Logger.log(sheet)
  var data = sheet.getDataRange().getValues();
  //Logger.log('data: ' + data)
  // 取得したデータをMapに変換。
  var sheetData = {};
  for (var i = 0; i < data.length; i++) {
    sheetData[data[i][0]] = {name : data[i][0], lastUpdate : data[i][1], rowNo : i + 1};
  }

  // 実際のフォルダとスプレッドシート情報を比較。
  var updateFolderMap = [];
  for (key in lastUpdateMap) {
    if( UPDATE_SHEET_ID == lastUpdateMap[key].fileId ){
      continue;
    }
    if(key in sheetData) {
      // フォルダ名がシートに存在する場合。
      if(lastUpdateMap[key].lastUpdate > sheetData[key].lastUpdate) {
        // フォルダが更新されている場合。
        sheet.getRange(sheetData[key].rowNo, 2).setValue(lastUpdateMap[key].lastUpdate);
        sheet.getRange(sheetData[key].rowNo, 3).setValue(lastUpdateMap[key].fileId);
        updateFolderMap.push({filename:key, lastUpdate:lastUpdateMap[key].lastUpdate, fileId:lastUpdateMap[key].fileId});
      }
    } else {
      // フォルダ名がシートに存在しない場合。
      var newRow = sheet.getLastRow() + 1;
      sheet.getRange(newRow, 1).setValue(key);
      sheet.getRange(newRow, 2).setValue(lastUpdateMap[key].lastUpdate);
      sheet.getRange(newRow, 3).setValue(lastUpdateMap[key].fileId);
      updateFolderMap.push({filename:key, lastUpdate:lastUpdateMap[key].lastUpdate, fileId:lastUpdateMap[key].fileId});
    }
  }
  //Logger.log('updateFolderMap:' + updateFolderMap)
  // 新規及び更新された情報をメール送信。
  var updateText = "";
  for( key in updateFolderMap ){
    item = updateFolderMap[key];
    updateText += 
     item.filename + '　更新日時：' + Utilities.formatDate(item.lastUpdate, "JST", "yyyy-MM-dd HH:mm:ss") + '\n' 
    + DriveApp.getFileById(item.fileId).getUrl() + "\n\n"
  }
  
  if (updateFolderMap.length != 0) {
    SEND_MAIL_ADDRESS.forEach(function(o,i) {
      MailApp.sendEmail(SEND_MAIL_ADDRESS[i],targetFolder.getName() + "更新連絡通知",
                        "【" + targetFolder.getName() + "】が更新されました。\n\n"+
                        updateText
                        );
    });
  }
}
