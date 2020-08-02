
---
マクロ

Sub Macro1()

    Dim wb As Workbook      'ワークブック
    Dim ws As Worksheet     'ワークシート
    Dim companyName As String
    Dim kiban As String
    Dim questionnaireDate As String
    Dim saveDir As String
    Dim fileName As String
    Dim userName As String
    
    '自ワークブック
    Set wb = ThisWorkbook
    'アクティブシート
    Set ws = ActiveSheet

    Worksheets(2).Copy

    Range("A1:C1").Select
    Range(Selection, Selection.End(xlDown)).Select
    Selection.Copy
    Selection.PasteSpecial Paste:=xlPasteAllUsingSourceTheme, Operation:=xlNone _
        , SkipBlanks:=False, Transpose:=False
    Selection.PasteSpecial Paste:=xlPasteValues, Operation:=xlNone, SkipBlanks _
        :=False, Transpose:=False
    Application.CutCopyMode = False
    
    Columns("D:O").Select
    Selection.Delete Shift:=xlToLeft
    
    companyName = ws.Range("C7").Value
    kiban = ws.Range("C13").Value
    questionnaireDate = Format(Left(ws.Range("C14").Value, 10), "yymmdd")
    
    Debug.Print (questionnaireDate)
    
 
    ' ログインユーザー名を取得する
    userName = CreateObject("WScript.Network").userName
    
    saveDir = "C:\Users\" + userName + "\Box\test" + "\" + Format(Date, "yyyymmdd")
    ' saveDir = "C:\Users\" + userName + "\Box\DEPT_カスタマーサポート部\150_満足度調査 お客様相談センター管理\web返信\" + Format(Date, "yyyymmdd")
    ' C:\Users\00183338\Box\DEPT_カスタマーサポート部\150_満足度調査 お客様相談センター管理\web返信
    fileName = companyName + "_" + kiban + "_" + questionnaireDate
    filePath = saveDir + "\" + fileName
    
    If Dir(saveDir, vbDirectory) = "" Then
        MkDir saveDir
    End If
    
    
    ActiveWorkbook.SaveAs filePath
    ActiveWorkbook.Close


End Sub
