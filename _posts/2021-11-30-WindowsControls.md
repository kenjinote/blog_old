---
layout: post
title: Windowsにおける標準コントロール一覧(C++)
thumbnail-img: /assets/img/windowscontrols.png
tags: [C++,UI,コントロール]
comments: true
---

### Windowsの標準コントロール一覧
Windowsの標準コントロールは以下になります。

- ボタンコントロール \"BUTTON\"
- コンボボックス \"COMBOBOX\"
- リストボックス \"LISTBOX\"
- MDI クライアントウィンドウ \"MDICLIENT\"
- スクロールバー \"SCROLLBAR\"
- スタティックコントロール \"STATIC\"
- リッチエディット 1.0 コントロール \"RichEdit\" ※Riched32.dllをロードしておく必要があります。
- リッチエディット 2.0 コントロール ANSI 版 \"RichEdit20A\" (RICHEDIT_CLASSA)
- リッチエディット 2.0 コントロール Unicode 版 \"RichEdit20W\" (RICHEDIT_CLASSA)

### Windowsのコモンコントロール一覧
コモンコントロールは下記になります。コントロールの作成前に InitCommonControls 関数または InitCommonControlsEx 関数を呼び出して、事前に使用するコントロールを登録しておく必要があります。

- アニメーションコントロール \"SysAnimate32\" (ANIMATE_CLASS)
- ホットキーコントロール \"msctls_hotkey32\" (HOTKEY_CLASS)
- プログレスバー \"msctls_progress32\" (PROGRESS_CLASS)
- ステータスバー \"msctls_statusbar32\" (STATUSCLASSNAME)
- ツールバー \"ToolbarWindow32\" (TOOLBARCLASSNAME)
- ツールチップコントロール \"tooltips_class32\" (TOOLTIPS_CLASS)
- トラックバー \"msctls_trackbar32\" (TRACKBAR_CLASS)
- アップダウンコントロール \"msctls_updown32\" (UPDOWN_CLASS)
- 拡張コンボボックス \"ComboBoxEx32\" (WC_COMBOBOXEX)
- ヘッダーコントロール \"SysHeader32\" (WC_HEADER)
- リストビュー \"SysListView32\" (WC_LISTVIEW)
- タブコントロール \"SysTabControl32\" (WC_TABCONTROL)
- ツリービュー \"SysTreeView32\" (WC_TREEVIEW)
- DTP コントロール \"SysDateTimePick32\" (DATETIMEPICK_CLASS) ※Comctl32.dll Version 4.70 以降
- 月間予定表コントロール \"SysMonthCal32\" (MONTHCAL_CLASS) ※Comctl32.dll Version 4.70 以降
- レバーコントロール \"ReBarWindow32\" (REBARCLASSNAME) ※Comctl32.dll Version 4.70 以降
- IPアドレスコントロール \"SysIPAddress32\" (WC_IPADDRESS) ※Comctl32.dll Version 4.71 以降
- ページャーコントロール　\"SysPager\" (WC_PAGESCROLLER) Comctl32.dll Version 4.71 以降

### その他のコントロール(クラス)
- DDEML イベントのクラス　\"DDEMLEvent\"
- メッセージ専用ウィンドウのクラス \"Message\"
- メニュークラス \"#32768\"
- デスクトップウィンドウクラス \"#32769\"
- ダイアログボックスクラス \"#32770\"
- タスクスイッチクラス \"#32771\"
- アイコンタイトルのクラス \"#32772\"

### 参考
- [About Window Classes](https://docs.microsoft.com/en-us/windows/win32/winmsg/about-window-classes)

