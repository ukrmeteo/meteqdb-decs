
```sql
// medb - Measuring Equipment DataBase


Table equip {                                    // measuring EQUIPment
  equ_id        int   [pk, increment]            // id обладнання (unsigned int: up to 16_777_215)
  equ_type      int   [ref: > type.typ_id]       // id обладнання (назва, маркировка)
  equ_numb      int                              // заводський номер
  equ_prod      year                             // рік виробництва (випуску)
  equ_state     enum('O','P','X')                // Основний / Резервний / Виведений з експлуатації
  equ_usedt     date                             // дата введення в експлуатацію (взяття на баланс)
  equ_setdt     date                             // дата встановлення на метеостанцію (змінюється після калібрування?)
  equ_bran      int   [ref: > branch.bra_id]     // id метео-станції (замовник)
  equ_caldt     date                             // дата останнього калібрування
  equ_fixdt     date                             // дата останнього ремонту (коли Виведений з експлуатації то це дата списання)
  updt          timestamp                        // дата оновлення інформації
}


Table etalon {                                  // Eталонні ЗВТ, які перебувають в експлуатації ЦГО
  eta_id        int    [pk, increment]          // id еталона (unsigned int: up to 16_777_215)5
  eta_type      int    [ref: > type.typ_id]     // id > назва еталона, маркировка
  eta_numb      int                             // заводський номер
  eta_prod      year                            // рік випуску
  eta_state     enum('O','X')                   // Основний / Виведений з експлуатації
  eta_usedt     date                            // дата введення в експлуатацію (взяття на баланс)
  eta_bran      int   [ref: > branch.bra_id]    // id метео-станції (замовник)
  eta_caldt     date                            // дата калібрування
  eta_fixdt     date                            // дата останнього ремонту (коли Виведений з експлуатації то це дата списання)
  updt          timestamp                       // дата оновлення
}


Table type {                                     // TYPEs of equipment
  typ_id        int      [pk, increment]         // id типу
  typ_name      varchar                          // назва гігрометр / анемометр / термометр
  typ_mark      varchar                          // маркировка М-19, ГС-210 / У-5, АСО-3 / КШ 14/23 (ТЛ-50)
  typ_cent      int      [ref: > center.cen_id]  // простежуваність до еталону (центру)
  mci_reco      int                              // МКІ міжкалібрувальний інтервал рекомендований, роки
  mci_base      int                              // МКІ міжкалібрувальний інтервал базовий, роки
  metro_ch      text                             // METROlogical CHaracteristics - Метрологічні характеристики обладнання
  updt          timestamp                        // дата оновлення
  Indexes {
    (typ_name, typ_mark)   [unique]
  }
}


Table method {                                   // методики
  met_id        int                              // id методу
  met_desc      vachar(256)                      // опис
  updt          timestamp                        // дата оновлення
}


Table calibr {                                   // калібрування "calibration"
  cal_id        int  [pk, increment]             // Реєстраційний номер з префіксом "99-04-32"
  cal_equi      int  [ref: > equip.equ_id]       // > назва, тип, зав/сер номер
  cal_meth      int  [ref: > method.met_id]      // методика
  cal_eta1      int  [ref: > etalon.eta_id]      // еталон 1
  cal_eta2      int  [ref: > etalon.eta_id]      // еталон 2 (для гигрометрів, другий термометр)
  cal_date      date                             // CALibration DaTe дата проведення калібрування
                                                 // Умови проведення калібрування - "Calibration conditions"
  temperat      float                            // TEMPERature температура  26.80С
  humidity      int                              // AIR HUMidity відносна вологість повітря  30%  ""
  pressure      float                            // ATMOspheric PRessure - атмосферний тиск  98.7 кПа
  manager       int [ref: > user.usr_id]         // MANAGER - керівник/менеджер - той, що прийняв і підписав
  assistent     int [ref: > user.usr_id]         // laboratory ASSISTant - той, що калібрував
  updt          timestamp                        // дата оновлення даних
}


Table trace {                                    // Переміщення обладнання
  tra_id        int    [pk, increment]           // id транзакції
  tra_equi      int    [ref: > equip.equ_id]     // id обладнання
  tra_user      int    [ref: > user.usr_id]      // хто? (користувач)
  tra_state     enum('S','B','O','K','P')        // дія: постановка / відправлення / отримання / калібрування / ремонт
  tra_bran      int    [ref: > branch.bra_id]    // куди? (підрозділ призначення)
  tra_comm      varchar                          // додатково
  updt          timestamp                        // дата оновлення даних
}


Table center {                                   // standardization CENTER - Центр стандартизації (простежуваність до еталону)
  cen_id        int    [pk, increment]           // TRACEability ID - ідентифікатор центра
  cen_name      varchar                          // NAME of Calibration centre - Назва калібрувального центру (Czech Calibration Station of Current Meters, ДП «УКРМЕТРТЕСТСТАНДАРТ»,+Харків})
  updt          timestamp                        // дата оновлення
}


Table branch {                                   // Centers for Hydrometeorology
  bra_id      int     [pk]                       // id (dd00 - центральні, dddd - їх підрозділи)
  bra_name    varchar                            //
  bra_addr    varchar                            //
  bra_tel1    varchar                            //
  bra_tel2    varchar                            //
  bra_email   varchar                            //
  updt        timestamp                          // дата оновлення
}


Table user {
  usr_id       int      [pk, increment]         //
  lname        varchar                          // Last NAME
  fname        varchar                          // First NAME
  sname        varchar                          // SurNAME
  tel1         varchar                          //
  tel2         varchar                          //
  email        varchar                          //
  passw        varchar                          // PASSWord
  posit        varchar                          // POSITion - посада
  usr_bran     int      [ref: > branch.bra_id]  // - підрозділ
  usr_state    int                              // стан
  usr_reg      date                             //
  updt         timestamp                        // дата оновлення
}


Table session {                                 // Таблиця "живих" сесій (історія усіх сесій - у лог-файл)
  ses_id       varchar                          // не unique - неймовірно, якщо співпаде, але якщо так, то все одно, окремо від користувача не береться
  ses_user     int      [unique, ref: - user.usr_id] //
  ses_keep     enum(0,1)                        // опція "запам'ятати мене" продовжує expire на 2 тижні при користуванні, зкидується розлогинуванням
  ses_exp      date                             // session "EXPiration date"
  updt         timestamp                        // дата оновлення
}

```