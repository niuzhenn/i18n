# autoUpdater

> Включает в приложении автоматическое самообновление.

Процесс: [Main](../glossary.md#main-process)

**See also: [A detailed guide about how to implement updates in your application](../tutorial/updates.md).**

## Платформа заметок

На текущий момент, поддерживается только macOS и Windows. Не существует встроенной поддержки для автоматического обновления на Linux, поэтому рекомендуется использовать дистрибутив пакетного менеджера для обновления Вашего приложения.

Кроме того есть некоторые тонкие различия на каждой платформе:

### macOS

На macOS `autoUpdater` модуль построен на [Squirrel.Mac](https://github.com/Squirrel/Squirrel.Mac), что означает, что вам не нужно каких-либо специальных настроек, чтобы заставить его работать. Для серверной стороны вы можете прочитать требования [Server Support](https://github.com/Squirrel/Squirrel.Mac#server-support). Заметьте, что [App Transport Security](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW35) (ATS) применяется ко всем запросам, как часть процесса обновления. Приложению, у которого требуется отключить ATS, добавьте ключ `NSAllowsArbitraryLoads` в список свойств приложения (property list).

**Примечание:** Ваше приложение должно быть подписано для автоматического обновления на macOS. Это требование `Squirrel.Mac`.

### Windows

На Windows, вам придется установить приложение на компьютер пользователя, прежде чем вы можете использовать `autoUpdater`, поэтому рекомендуется использовать пакет [electron-winstaller](https://github.com/electron/windows-installer), [electron-forge](https://github.com/electron-userland/electron-forge) или [grunt-electron-installer](https://github.com/electron/grunt-electron-installer) для создания установщика Windows.

При использовании [electron-winstaller](https://github.com/electron/windows-installer) или [electron-forge](https://github.com/electron-userland/electron-forge) убедитесь, что вы не пытаетесь обновить ваше приложение [при первом запуске](https://github.com/electron/windows-installer#handling-squirrel-events) (также см. [этот вопрос для получения дополнительной информации](https://github.com/electron/electron/issues/7155)). Также рекомендуется использовать [electron-squirrel-startup](https://github.com/mongodb-js/electron-squirrel-startup) для получения ярлыка рабочего стола для вашего приложения.

Установщик сгенерирует Squirrel создаст ярлык с [Application User Model ID](https://msdn.microsoft.com/en-us/library/windows/desktop/dd378459(v=vs.85).aspx) в формате `com.squirrel.PACKAGE_ID.YOUR_EXE_WITHOUT_DOT_EXE`, `com.squirrel.slack.Slack` и `com.squirrel.code.Code`. Вы должны использовать тот же ID для вашего приложения с `app.setAppUserModelId` API, в противном случае Windows не сможет должным образом закрепить приложение в панели задач.

В отличие от Squirrel.Mac, Windows обновления можно размещать на S3 или любом другом хостинге статических файлов. Вы можете прочитать документы [Squirrel.Windows](https://github.com/Squirrel/Squirrel.Windows) для получения более подробной информации о том, как работает Squirrel.Windows.

## События

Объект `autoUpdater` имеет следующие события:

### Событие: 'error'

Возвращает:

* `error` Error

Возникает когда происходит ошибка при обновлении.

### Событие: 'checking-for-update'

Возникает при проверке, если обновление началось.

### Событие: 'update-available'

Возникает при наличии доступных обновлений. Обновление загружается автоматически.

### Событие: 'update-not-available'

Возникает, когда нет доступных обновлений.

### Событие: 'update-downloaded'

Возвращает:

* `event` Event
* `releaseNotes` String
* `releaseName` String
* `releaseDate` Date
* `updateURL` String

Возникает при загрузке обновления.

На Windows доступен только `releaseName`.

**Note:** It is not strictly necessary to handle this event. A successfully downloaded update will still be applied the next time the application starts.

### Event: 'before-quit-for-update'

This event is emitted after a user calls `quitAndInstall()`.

When this API is called, the `before-quit` event is not emitted before all windows are closed. As a result you should listen to this event if you wish to perform actions before the windows are closed while a process is quitting, as well as listening to `before-quit`.

## Методы

Объект `autoUpdater` имеет следующие методы:

### `autoUpdater.setFeedURL(options)`

* `options` Object 
  * `url` String
  * `headers` Object (опционально) *macOS* - заголовки запросов HTTP.
  * `serverType` String (опционально) *macOS* - Любой `json` или `default`, смотри [Squirrel.Mac](https://github.com/Squirrel/Squirrel.Mac) README для подробностей.

Задает `url` и инициализирует автоматическое обновление.

### `autoUpdater.getFeedURL()`

`String` - возвращает URL текущей подписки обновления.

### `autoUpdater.checkForUpdates()`

Запрашивает сервер на наличие обновлений. Перед использованием этого API-интерфейса, необходимо вызвать `setFeedURL`.

### `autoUpdater.quitAndInstall()`

Перезапускает приложение и устанавливает обновления после того как скачает. Должен вызываться только после того, как возникнет событие `update-downloaded`.

Внутри, вызов `autoUpdater.quitAndInstall()` сначала закроет все окна приложения, и автоматически вызовет `app.quit()`, после того, как все окна будут закрыты.

**Note:** It is not strictly necessary to call this function to apply an update, as a successfully downloaded update will always be applied the next time the application starts.