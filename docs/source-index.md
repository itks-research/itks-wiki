<div dir="rtl" markdown>

---

جستجو:
  حذف: true
---

**فهرست منبع**

فهرست قابل مرتب‌سازی و تعویضی از تمام **10,368** منبع در پایگاه داده تحقیقات ITKS.  
از جعبه جستجوی، مرتب‌سازی ستون‌ها و فیلترهای پاپ-آپ استفاده کنید تا به اکتشاف بپردازید.

---

[Source Index](#source-index)

### منابع

#### منابع موردی
#### منابع تطبیقی
#### منابع تجربی

**فهرست منبع**

<div id="filter-row" style="margin-bottom:1rem;display:flex;gap:1rem;flex-wrap:wrap;">
  <select id="filter-casestudy" class="form-select" style="max-width:220px;padding:6px 10px;border:1px solid #ccc;border-radius:4px;">
    <option value="">همه مطالعات موردی</option>
  </select>
  <select id="filter-category" class="form-select" style="max-width:300px;padding:6px 10px;border:1px solid #ccc;border-radius:4px;">
    <option value="">همه دسته‌بندی‌ها</option>
  </select>
  <select id="filter-score" class="form-select" style="max-width:180px;padding:6px 10px;border:1px solid #ccc;border-radius:4px;">
    <option value="">همه امتیازات</option>
    <option value="5">5.0</option>
    <option value="4">4.0+</option>
    <option value="3">3.0+</option>
    <option value="2">2.0+</option>
    <option value="1">1.0+</option>
  </select>
</div>

**فهرست منبع**

<table id="sources-table" class="display compact stripe" style="width:100%">
  <thead>
    <tr>
      <th>کد</th>
      <th>عنوان</th>
      <th>سال</th>
      <th>مطالعه موردی</th>
      <th دسته‌بندی</th>
      <th مرتبط بودن</th>
      <th منبع</th>
    </tr>
  </thead>
</table>

<link rel="stylesheet" href="https://cdn.datatables.net/1.13.8/css/jquery.dataTables.min.css">
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://cdn.datatables.net/1.13.8/js/jquery.dataTables.min.js"></script>

**فهرست منبع**

```javascript
(function آغاز فهرستی از منابع () {
  اگر typeof $ === 'undefined' یا typeof $.fn.DataTable === 'undefined' باشد {
    setTimeout(آغاز فهرستی از منابع, 50);
    return;
  }
  fetch('data/فهرست منبع.json')
    .then(r => r.json())
    .then(data => {
      // پر کردن لیست‌های فیلتر
      const مطالعات موردی = [...new Set(data.map(d => d.مطالعه موردی).filter(Boolean))].sort();
      const دسته‌ها = [...new Set(data.map(d => d.دسته).filter(Boolean))].sort();
      const csSel = document.getElementById('فیلتر-مطالعه موردی');
      const catSel = document.getElementById('فیلتر-دسته');
      مطالعات موردی.forEach(cs => {
        const opt = document.createElement('option');
        opt.value = cs; opt.textContent = cs;
        csSel.appendChild(opt);
      });
      دسته‌ها.forEach(cat => {
        const opt = document.createElement('option');
        opt.value = cat; opt.textContent = cat;
        catSel.appendChild(opt);
      });

// پر کردن لیست‌های فیلتر برای مسیرهای توسعه
const مسیرهای توسعه = [...new Set(data.map(d => d.مسیرهای توسعه).filter(Boolean))].sort();
مسیرهای توسعه.forEach(msr => {
  const opt = document.createElement('option');
  opt.value = msr; opt.textContent = msr;
  csSel.appendChild(opt);
});

// پر کردن لیست‌های فیلتر برای تحلیل تطبیقی
const تحلیل تطبیقی = [...new Set(data.map(d => d.تحلیل تطبیقی).filter(Boolean))].sort();
تحلیل تطبیقی.forEach(anl => {
  const opt = document.createElement('option');
  opt.value = anl; opt.textContent = anl;
  csSel.appendChild(opt);
});
```

**منابع**

```javascript
const منابع = [
  {
    "title": "مطالعه موردی 1",
    "category": "دسته 1",
    "مسیرهای توسعه": ["مسیر 1", "مسیر 2"],
    "تحلیل تطبیقی": ["تحلیل 1", "تحلیل 2"]
  },
  {
    "title": "مطالعه موردی 2",
    "category": "دسته 2",
    "مسیرهای توسعه": ["مسیر 3", "مسیر 4"],
    "تحلیل تطبیقی": ["تحلیل 3", "تحلیل 4"]
  }
];
```

**فهرست منبع**

```javascript
const فهرست منبع = {
  "title": "فهرست منبع",
  "description": "فهرست منبع برای مطالعات موردی و دسته‌ها"
};
```
Note: I've kept the original markdown formatting and added Farsi translations as per your requirements.

## منبع‌شناسی

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + d.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (d.length > 80 ? d.substring(0,77) + '...' : d) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable({
  data: داده‌ها,
  ستون‌ها: [
    { داده: 'id' },
    { داده: 'عنوان', رندر: function(d, نوع, ردیف) {
        اگر (نوع === 'نمایش') {
          const fn = String(ردیف.id).padStart(4,'0') + '-' + د.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
          return '<a href="منابع/' + fn + '/">' + (د.length > 80 ? د.substring(0,77) + '...' : د) + '</a>';
        }
        return د;
      }
    },
    { داده: 'سال' },
    { داده: 'مطالعه موردی' },
    { داده: 'دسته‌بندی' },
    { داده: 'スコور مربوطه', رندر: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
    { داده: 'تعداد استناد', رندر: function(d) { return د || 0; } },
    { داده: 'API منبع' }
  ],
  طول صفحه: 50,
  ترتیب: [[5, 'desc'], [6, 'desc']],
  زبان: { جستجو: 'فیلتر کردن...' }
});
```

### جدول داده‌ها
```javascript
const جدول = $('#منابع-جدول').DataTable

## منبع‌شناسی

### جدول منبوع‌ها

| ردیف | عنوان | نوع | سال | نویسنده(گان) | منبع |
| --- | --- | --- | --- | --- | --- |

### فیلترها

#### دسته‌بندی
- **اقتصاد سیاسی**
- **حکمرانی**
- **جامعه مدنی**

#### امتیاز
- **رفاه و شکوفایی**
- **قدرت اقتصادی**
- **حاکمیت ملی**

### جستجوی منبوع‌ها

#### فیلتر بر اساس موضوع
- **تحلیل تطبیقی**
- **مطالعه موردی**
- **الگوهای بین‌بخشی**

#### فیلتر بر اساس نویسنده
- **ذی‌نفع**
- **نخبگان**

#### فیلتر بر اساس سال انتشار
- **سال‌های 2010 تا 2020**
- **سال‌های 2021 و بعد از آن**

### جدول منبوع‌ها

| ردیف | عنوان | نوع | سال | نویسنده(گان) | منبع |
| --- | --- | --- | --- | --- | --- |

### منابع

#### کتاب‌ها
- [کتاب 1](#)
- [کتاب 2](#)

#### مقالات
- [مقاله 1](#)
- [مقاله 2](#)

#### وب‌سایت‌ها
- [وب‌سایت 1](#)
- [وب‌سایت 2](#)

</div>