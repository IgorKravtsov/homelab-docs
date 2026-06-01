# TrueNAS Compact Build — $500-700 Budget

## Требования

- Компактный корпус (не как полноразмерный ПК)
- Mini-ITX форм-фактор
- TrueNAS SCALE совместимость
- Бюджет: $500-700 (без дисков)

---

## Вариант 1: DIY сборка (Mini-ITX)

### Комплектующие

| Компонент | Модель | Цена | Ссылка |
|-----------|--------|------|--------|
| **Корпус** | Fractal Design Node 304 (6x 3.5" bays) | $60-70 (б/у eBay) | [eBay](https://www.ebay.com/itm/301380742683) |
| **Корпус** | SilverStone DS380 (8x 3.5" hot-swap) | $50-60 (б/у) | [Newegg](https://www.newegg.com/p/2KH-0030-001S4) |
| **CPU** | Intel i5-8500 (6 cores, 65W) | $30-40 | [eBay](https://www.ebay.com/itm/353854457929) |
| **Motherboard** | ASRock B365M-ITX/ac (Mini-ITX, LGA1151) | $50-60 | [eBay](https://www.ebay.ca/itm/165824766610) |
| **RAM** | 16GB DDR4 (non-ECC) | $25-30 | [eBay](https://www.ebay.com/b/16GB-Computer-DDR4-SDRAM/170083/bn_90996560) |
| **RAM** | 32GB DDR4 ECC | $50-70 | eBay — искать "DDR4 ECC 32GB" |
| **Boot SSD** | 64GB SATA SSD | $15-20 | Amazon / AliExpress |
| **PSU** | 350W 80+ Bronze | $30-40 | [eBay](https://www.ebay.com/itm/405399937719) |

### Итого (бюджетный вариант)

| Компонент | Цена |
|-----------|------|
| Корпус (б/у) | $60 |
| CPU i5-8500 | $35 |
| MBO B365M-ITX | $55 |
| RAM 16GB | $25 |
| Boot SSD | $15 |
| PSU 350W | $30 |
| **ИТОГО** | **~$220** |

**С ECC RAM:** ~$260-300

---

## Вариант 2: Готовый Mini-PC + внешнее хранение дисков

### Dell OptiPlex 7060 Micro

| Параметр | Значение |
|----------|----------|
| **Форм-фактор** | Ultra-small (0.6L) — очень компактный |
| **CPU** | i5-8500T (6 cores, 1.6-3.5GHz, 35W TDP) |
| **RAM** | 16GB DDR4 (до 64GB) |
| **Storage** | 256GB NVMe SSD |
| **Сеть** | Gigabit Ethernet |
| **Цена** | $180-250 (refurbished) |

**Где купить:**
- [discountelectronics.com](https://discountelectronics.com/dell-optiplex-7060-micro-i5-16gb-ram-ssd-windows-11-pro-desktop-tiny-pc/) — $200-250
- [Amazon](https://www.amazon.com/Dell-OptiPlex-7060-Micro-i5-8500T/dp/B0G459Z7F7) — ~$250
- [refurb.io](https://us.refurb.io/products/dell-optiplex-7060-micro-desktop-intel-i5-8500t-1-6-ghz-16gb-512gb-ssd-windows-11-pro-refurbished) — ~$220

### HP ProDesk 600 G4 Mini

| Параметр | Значение |
|----------|----------|
| **Форм-фактор** | Mini Desktop (0.9L) |
| **CPU** | i5-8500T (6 cores) |
| **RAM** | 16GB DDR4 |
| **Цена** | $150-220 |

**Где купить:**
- [eBay](https://www.ebay.com/itm/226586808887) — ~$180-200
- [Amazon](https://www.amazon.com/HP-600-G4-i5-8500T-Computer/dp/B0FB4B7GRG) — ~$200

---

## Вариант 3: Внешнее хранение дисков (для Mini-PC)

### USB Docking Station (1-bay)

| Товар | Цена | Ссылка |
|-------|------|--------|
| **Sabrent USB 3.0 Docking** (1-bay) | $20-25 | [Sabrent](https://sabrent.com/products/ec-ublb) |
| **StarTech Single Bay** (2.5/3.5") | $55-70 | [StarTech](https://www.startech.com/en-us/hdd/satdocku3s) |
| **ORICO 1-bay USB 3.0** | $15-20 | [Newegg](https://www.newegg.com/orico-3588us3-us-bk-enclosure/p/0VN-0003-000W8) |

### External RAID Enclosure (4-bay)

| Товар | Цена | Ссылка |
|-------|------|--------|
| **ORICO 4-bay RAID** (USB 3.0) | $120-150 | [Amazon](https://www.amazon.com/ORICO-External-Enclosure-Supports-9948RU3/dp/B0DGD1X36L) |
| **ORICO 4-bay** (new) | $200 | [Newegg](https://www.newegg.com/orico-9858ru3-us-gy-p-dock/p/0VN-0003-002Z8) |
| **Unitek 4-bay** | $100-150 | [Unitek](https://www.unitek-products.com/products/4-bay-raid-external-hard-drive-enclosure) |

### Готовые external HDD

| Товар | Цена | Ссылка |
|-------|------|--------|
| **WD Elements 4TB** | $80-100 | Amazon |
| **Seagate Expansion 4TB** | $80-100 | Amazon |
| **WD Elements 8TB** | $130-150 | Amazon |

---

## Рекомендация

### Для максимальной компактности
**Dell OptiPlex 7060 Micro** + **2 x USB docks** = ~$250 + $40 = **~$290** (без дисков)

### Для полноценного NAS
**DIY Node 304** = ~$220-300 (без дисков)

### С дисками (2x4TB mirror)

| Вариант | Стоимость с дисками |
|---------|---------------------|
| OptiPlex + 2x USB dock + 2x4TB | ~$290 + $160 = **~$450** |
| DIY Node 304 + 2x4TB | ~$260 + $160 = **~$420** |

---

## Важно знать

### TrueNAS требования

| Компонент | Минимум | Рекомендуется |
|-----------|---------|---------------|
| CPU | 2 cores | 4+ cores |
| RAM | 8 GB | 32 GB ECC |
| Boot | 16 GB | 32+ GB SSD |
| Сеть | 1 Gbe | 2.5-10 Gbe |

### USB подключение дисков

- TrueNAS работает по USB, но **нет hot-swap**
- Медленнее чем SATA (но для NAS это не критично)
- Лучше использовать eSATA если есть порт

### Intel i5-8500 и ECC память

**Важно:** i5-8500 **не поддерживает ECC память**. Это десктопный процессор, а не серверный Xeon.

| CPU | ECC Support |
|-----|--------------|
| Intel i5-8500 | ❌ Нет |
| Intel Xeon E-2224 | ✅ Да |

Для ECC нужно:
- **CPU:** Intel Xeon (E-2100, E-2200 series)
- **Motherboard:** Чипсет C246 / C232 (Supermicro)

**TrueNAS на non-ECC** — работает, но меньше защита от bit-flip ошибок в памяти.

```
2 × 4TB WD Ultrastar DC HC310 → Mirror → 4TB полезного
```

**Почему Mirror:**
- Просто настроить
- Надежно (1 диск может умереть)
- Легко расширять (добавить новый vdev)

---

## Ссылки

- TrueNAS Hardware Guide: https://www.truenas.com/docs/scale/25.04/gettingstarted/scalehardwareguide/
- Best Hardware for TrueNAS 2026: https://selfhosting.sh/hardware/truenas-hardware-guide/
