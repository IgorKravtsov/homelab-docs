# TrueNAS Build — Бюджетный вариант

## ✅ Выбранные комплектующие (Вариант 2: Середнячок с IPMI)

| Компонент | Модель | Цена (Польша) |
|-----------|--------|--------------|
| **CPU** | Intel Xeon E-2224 (4 cores, 3.4GHz, LGA 1151) | ~$50-70 (б/у Ali) |
| **MBO** | Supermicro X11STL-iF (IPMI, 8 SATA, C246) | ~$80-100 (б/у Ali) |
| **RAM** | Kingston DDR4 16GB ECC UDIMM | ~700-860 zł (новый) |
| **Boot SSD** | 64GB SATA SSD (Kingston/Crucial/Samsung) | ~150-200 zł (новый) |
| **HDD** | 2x Seagate IronWolf 4TB (7200 RPM) | ~1000-1200 zł (новый) |
| **Корпус** | Fractal Design Node 304 (Mini-ITX, 6x 3.5" bays) | ~320 zł (новый) |
| **Кулер CPU** | Arctic Alpine 20 Plus / Noctua NH-L9i | ~100-150 zł |
| **Блок питания** | 350-450W 80+ Bronze (Corsair CX, Seasonic) | ~200-300 zł |
| **SATA кабели** | 2x SATA 6Gb/s (0.5m, с защелкой) | ~20-40 zł |

**Примечание:** Требуется прошивка BIOS для поддержки Xeon E-2224.

---

## 📊 Итого

| Категория | Цена |
|-----------|------|
| CPU + MBO (б/у) | ~$130-170 |
| RAM (новый) | ~700-860 zł |
| Boot SSD (новый) | ~150-200 zł |
| HDD x2 (новый) | ~1000-1200 zł |
| Корпус (новый) | ~320 zł |
| Кулер CPU (новый) | ~100-150 zł |
| БП 350-450W (новый) | ~200-300 zł |
| SATA кабели | ~20-40 zł |
| **Всего** | **~$900-1100** |

---

## ZFS и RAID конфигурация

### Термины

| Термин | Описание |
|--------|----------|
| **Pool** | Всё хранилище (аналог LVM) |
| **vdev** | Виртуальное устройство (группа дисков) |
| **Mirror** | RAID1 — зеркало (2 копии данных) |
| **RAIDZ1** | RAID5 — 1 parity, минимум 3 диска |
| **RAIDZ2** | RAID6 — 2 parity, минимум 4 диска |

### Типы vdev

```
┌─────────────────────────────────────────────────────┐
│                     ZFS Pool                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │   vdev 1    │  │   vdev 2   │  │   vdev 3    │  │
│  │  (mirror)   │  │  (raidz1)  │  │  (raidz2)   │  │
│  │ ┌─┐   ┌─┐   │  │ ┌─┐┌─┐┌─┐  │  │ ┌─┐┌─┐┌─┐┌─┐│  │
│  │ │D│   │D│   │  │ │D││D││D│  │  │ │D││D││D││D│  │
│  │ └─┘   └─┘   │  │ └─┘└─┘└─┘  │  │ └─┘└─┘└─┘└─┘│  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Сравнение RAID

| Тип | Мин. дисков | Полезное (4×4TB) | Защита |
|-----|-------------|------------------|--------|
| **Mirror** | 2 | 4 TB (50%) | 1 диск |
| **RAIDZ1** | 3 | 8 TB (67%) | 1 диск |
| **RAIDZ2** | 4 | 8 TB (50%) | 2 диска |
| **2× Mirror** | 4 | 8 TB (50%) | 2 диска (независимые!) |

### Рекомендации для дома

| Дисков | Рекомендация | Полезное | Защита |
|--------|--------------|----------|--------|
| 2 | Mirror | 4 TB | 1 |
| 3 | RAIDZ1 | 8 TB | 1 |
| 4 | 2× Mirror | 8 TB | 2 ✅ |
| 5 | RAIDZ1 | 12 TB | 1 |
| 6+ | RAIDZ2 | varies | 2+ |

### Наш выбор: Старт с 2× Mirror

```
2 × 4TB Seagate IronWolf → Mirror → 4TB полезного
```

**Почему Mirror:**
- Просто настроить
- Надежно (1 диск может умереть)
- Легко расширять

---

## Масштабирование (Expansion)

### Как добавлять диски

| Добавить | Минимум | Результат |
|----------|---------|-----------|
| +2 диска | 2 (mirror) | +4TB |
| +3 диска | 3 (raidz1) | +8TB |
| +4 диска | 4 (mirror/raidz2) | +4-8TB |

### Пример расширения пула

```
Старт (2 диска):
Pool "storage"
  └─ vdev-1 (mirror)
      ├─ /dev/sda (4TB)
      └─ /dev/sdb (4TB)
→ 4TB

После +2 диска:
Pool "storage"
  ├─ vdev-1 (mirror)      ← старые
  │   ├─ /dev/sda
  │   └─ /dev/sdb
  └─ vdev-2 (mirror)      ← новые
      ├─ /dev/sdc
      └─ /dev/sdd
→ 8TB полезного
```

### Важно: нельзя менять тип vdev

| Из | В | Возможно? |
|----|---|-----------|
| Mirror | RAIDZ1 | ❌ |
| RAIDZ1 | RAIDZ2 | ❌ |
| Добавить новый vdev | ✅ |

**Решение:** Добавляешь новый vdev с другой конфигурацией.

---

## Смешанные vdev ( пример)

```
Pool "storage"
  ├─ vdev-1 (mirror)      ← быстрый, для активных файлов
  └─ vdev-2 (raidz2)      ← большой, для архивов
```

Все приложения (Nextcloud, SMB, Plex) **видят единое хранилище** — не важно, какой там внутри vdev.

---

## Выводы

1. **Старт:** 2×4TB в Mirror (4TB полезного)
2. **Расширение:** Добавлять по 2+ диска как новые vdev
3. **Гибкость:** Можно смешивать mirror/raidz1/raidz2 в одном пуле
4. **Приложения:** Видят один пул "storage" — не заботятся о конфиге

---

## Конфигурация (~$250-350)

| Компонент | Модель | Цена (б/у) | Где искать |
|-----------|--------|------------|------------|
| **Корпус** | Fractal Design Node 304 или SilverStone DS380 | $40-60 | eBay, Reddit |
| **CPU** | Intel i5-8500 (6 cores, 65W) | $30-40 | eBay |
| **Motherboard** | Dell OptiPlex 7060 Tower (встроенный CPU) | $120-150 | eBay |
| **RAM** | 32 GB DDR4 ECC (2x16GB) | $40-60 | eBay, AliExpress |
| **Boot SSD** | 64 GB SATA SSD | $15-20 | Amazon |
| **Диски** | 4x 4 TB Seagate IronWolf | $80-100 (б/у) | eBay |
| **HBA** | LSI 9207-8i (IT mode) | $25-35 | eBay |

**Итого: ~$250-350**

---

## Почему Dell OptiPlex 7060 Tower

- i5-8500 (6 cores, 3.0 GHz) — 65W TDP
- Поддерживает до 64 GB RAM
- Встроенный блок питания
- Компактный и тихий для дома
- Легко найти б/у

---

## Требования к компонентам

### RAM

- Тип: DDR4 ECC (UDIMM, не RDIMM)
- Объём: минимум 16 GB, рекомендуется 32 GB
- Производители: Kingston, Samsung, Micron

### HBA (Host Bus Adapter)

- Модель: LSI 9207-8i
- **Важно**: Прошивка в IT mode (не IR)
- IT mode = JBOD, ZFS управляет дисками напрямую
- Без RAID — сам RAID делает ZFS

### Диски

- Тип: NAS-диски (7200 RPM)
- Рекомендуются: Seagate IronWolf, WD Red Plus
- Перед покупкой: проверь SMART статус
- Старт: 4x 4TB в RAIDZ1 (~10.5 TB полезного)

---

## Где покупать б/у

1. **eBay** — основной источник
2. **r/homelabsales** (Reddit) — распродажи сообщества
3. **Facebook Marketplace** — локальные предложения
4. **AliExpress** — RAM (проверь отзывы)

---

## Альтернативная конфигурация: HP ProLiant DL360 Gen10

| Компонент | Описание |
|-----------|----------|
| CPU | Xeon Silver 4214R |
| RAM | 64+ GB ECC |
| Bays | 8x 3.5" hot-swap |
| Цена | $200-300 (б/у) |

Плюсы: enterprise-grade, hot-swap диски
Минусы: шумнее, больше энергопотребление

---

## Расчёт хранилища

| Конфигурация | Полезное хранилище |
|--------------|-------------------|
| 4x 4 TB (RAIDZ1) | ~10.5 TB |
| 4x 8 TB (RAIDZ1) | ~21 TB |
| 6x 8 TB (RAIDZ2) | ~32 TB |

---

## Несколько пулов: защищенные и незащищенные данные

### Зачем нужно

Можно разделить важные и неважные данные:
- **Mirror** — для важных документов
- **Single (без RAID)** — для мусора, временных файлов

### Пример реализации

```
Pool "protected"   (mirror)     → /mnt/protected    ← Важные документы
Pool "unprotected" (single)     → /mnt/unprotected  ← Неважные данные
```

### Использование в приложениях

| Приложение | Путь | Пул |
|------------|------|-----|
| Nextcloud Documents | `/mnt/protected/nextcloud/docs` | protected |
| Nextcloud Temp | `/mnt/unprotected/nextcloud/temp` | unprotected |
| SMB Documents | `/mnt/protected/documents` | protected |
| SMB Media | `/mnt/unprotected/media` | unprotected |

### Расширение

```
Старт (2×4TB):
Pool "main" → 2×4TB Mirror → 4TB (защищено)

Потом (+8TB):
Pool "main"     → 2×4TB Mirror → 4TB (защищено)
Pool "bulk"     → 1×8TB        → 8TB (незащищено)
```

---

## Требования TrueNAS SCALE

| Компонент | Минимум | Рекомендуется |
|-----------|---------|---------------|
| CPU | 2 cores | 4+ cores |
| RAM | 8 GB | 32 GB ECC |
| Boot | 16 GB | 32+ GB SSD |
| Сеть | 1 Gbe | 2.5-10 Gbe |

---

## Ссылки

- TrueNAS Hardware Guide: https://www.truenas.com/docs/scale/25.04/gettingstarted/scalehardwareguide/
- Best Hardware for TrueNAS 2026: https://selfhosting.sh/hardware/truenas-hardware-guide/
- 100TB NAS Build Guide: https://techlife.blog/posts/build-your-own-100tb-nas-2025-complete-truenas-storage-guide/