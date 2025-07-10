# Zabbix Template: DCM4CHE 2.x MWL Service Availability

## Описание

Этот шаблон предназначен для мониторинга **доступности MWL-сервиса (Modality Worklist)** в PACS-системе **DCM4CHE 2.x** с помощью Zabbix.

Шаблон проверяет отклик MWL-сервиса по протоколу DICOM через внешнюю проверку (например, `dcmmwl`/`dcm4che-utils`).

## Содержимое шаблона

- Элемент данных: проверка доступности MWL-сервиса (внешний скрипт)
- Триггер: "MWL service is unavailable"
- (опционально) График отклика

## Требования

- **Zabbix версии:** 5.0
- **DCM4CHE:** версия 2.x (например, 2.0.17)
- **DICOM tools:** dcm4che CLI утилита (`dcmmwl`)
- **Agent и External Script:** скрипт, который проверяет MWL запрос

## Установка

1. Импортируйте файл `template_dcm4che_mwl.xml` в Zabbix:
   - `Configuration → Templates → Import`
2. Примените шаблон к хосту, где работает MWL-сервис.
3. При необходимости укажите:
   - DICOM AE Title
   - DICOM IP и порт
   - Локальные пути к утилитам

## Использование скрипта который выполнит запрос и подсчитает первые 3 ответа, содержащие тег PatientName (0010,0010)

Скрипт может выглядеть так (например: `check_mwl.sh`):

<details>
<summary>Показать код check_mwl.sh</summary>

```bash   
#!/bin/bash

MWLSCP="DCM4CHEE@127.0.0.1:11112"
CALLING_AET="ZABBIX_CHECK"
DCMMWL_BIN="/path_to_dcmutils/dcmmwl"
TODAY="$(date +%Y%m%d)"
MATCHED_PATIENTS=$("$DCMMWL_BIN" "$MWLSCP" \
  -L "$CALLING_AET" \
  -date "$TODAY-$TODAY" \
  -r PatientName 2>/dev/null | grep "(0010,0010)" | head -n 3 | wc -l)

if [ "$MATCHED_PATIENTS" -gt 0 ]; then
  echo 1
else
  echo 0
fi
```
</details>

Сделать скрипт исполняемым chmod +x check_mwl.sh

## Внесение в конфигурационный файл Zabbix-агента параметра

UserParameter=mwl.check,/path_to_script/check_mwl.sh
