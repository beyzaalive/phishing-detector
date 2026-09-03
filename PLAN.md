# Phishing / Dolandırıcılık Tespit Sistemi — Proje Planı

## Amaç
Şüpheli bir SMS, e-posta ya da link metnini analiz edip, bunun bir dolandırıcılık/phishing girişimi olup olmadığını **anlaşılır bir dille** açıklayan, kural tabanlı bir tespit sistemi.

## Problem
Türkiye'de yaygın dolandırıcılık kalıpları (sahte kargo SMS'i, sahte banka bildirimi, sahte e-Devlet uyarısı vb.) hızla değişiyor ve herkes (teknik bilgisi olan biri bile) bir anlık dikkatsizlikle kanabiliyor. VirusTotal, Kaspersky gibi mevcut araçlar var ama genel/global odaklı, Türkçe'ye özgü dilbilimsel/kültürel kalıplara bakmıyor.

## Hedef Kullanıcı
Şüphelenen ama emin olamayan kişiler, ve/veya yakınları adına kontrol eden farkındalığı yüksek kullanıcılar. (Hiç şüphelenmeden direkt tıklayan kullanıcıya bu tür bir araç zaten ulaşamaz — bu bilinen bir sınırlama, README'de belirtilecek.)

## Kapsam Dışı (bilinçli tercih)
- Mesajı doğrudan bir LLM'e verip "bu phishing mi?" diye sormak → prompt injection riski taşır, ayrıca öğretici değil
- Üretim seviyesinde, gerçek kullanıcı trafiği taşıyacak bir servis kurmak → bu bir portföy/öğrenme projesi
- Hiç şüphelenmeyen kullanıcıya ulaşmayı çözmek → yazılımla çözülemeyecek bir problem, kapsam dışı
- Şimdilik ML/AI eklemek → önce kural tabanlı sistemi tamamen anlayarak ve kendin geliştirerek tamamla; ML/AI ileride ayrı bir "gelecek çalışma" olabilir

## Mimari (özet)
```
Girdi (SMS/e-posta metni)
        │
        ▼
   ┌────┴────┬──────────┐
   ▼         ▼          ▼
URL modülü  Domain    Metin modülü
(bulgu)     modülü    (bulgu)
            (bulgu)
   │         │          │
   └────┬────┴──────────┘
        ▼
Risk motoru (bulguları toplar, skorlar, nedenleri listeler)
        │
        ▼
API (FastAPI)
        │
        ▼
Arayüz (web sayfası → ileride Telegram botu)
```

## Analiz Motoru Tasarım İlkeleri
- **Feature ≠ verdict:** Hiçbir modül tek başına "bu phishing" demez — her modül sadece kendi alanındaki bir **bulguyu** (finding) üretir (örn. "IP tabanlı link", "domain 12 gün önce kayıtlı", "aciliyet dili tespit edildi").
- **Nihai kararı sadece risk motoru verir** — tüm modüllerden gelen bulguları toplayıp bir risk skoru + risk seviyesi (düşük/orta/yüksek) + nedenler listesi üretir.
- **Sistem "kesin güvenli" garantisi vermez** — sonuç her zaman "gözlemlenen göstergelere göre" diye açıklanabilir biçimde sunulur. False positive/negative olabileceği kabul edilir; README'de bu açıkça belirtilecek: *bu bir risk değerlendirme aracıdır, kesin güvenlik garantisi vermez.*

## Fazlar

**Faz 1 — Çekirdek (şu an buradayız)**
- [x] Git kuruldu, repo GitHub'a bağlandı

*URL modülü (`url_analiz.py`)*
- [ ] IP tabanlı link tespiti *(şu an bu adımdayız)*
- [ ] HTTPS var mı yok mu
- [ ] Domain/subdomain ayrıştırma (aldatıcı subdomain tespiti, örn. `google.com.fake-site.com`)
- [ ] Şüpheli karakterler, aşırı uzun URL, şüpheli parametreler

*Domain modülü (`domain_analiz.py`)*
- [ ] Domain yaşı (sinyal olarak — kesin karar değil)
- [ ] Bilinen kötü amaçlı liste eşleşmesi (opsiyonel, dış kaynağa bağlı)

*Metin modülü (`metin_analiz.py`)* — kategori kategori, tek seferde hepsi değil
- [ ] Aciliyet kalıpları (önce bununla başla)
- [ ] Tehdit kalıpları
- [ ] Ödül/para kalıpları
- [ ] Kimlik/doğrulama isteği kalıpları
- [ ] Genel sosyal mühendislik kalıpları

*Risk motoru (`risk_motoru.py`)*
- [ ] Modüllerden gelen bulguları toplama
- [ ] Ağırlıklandırma/skorlama mantığı
- [ ] Risk seviyesi + nedenler çıktısı

*Test fazı*
- [ ] Her modül için ayrı testler (pytest)
- [ ] Güvenli / şüpheli / sınırda örneklerden oluşan küçük test seti
- [ ] Edge case'ler: aldatıcı subdomain, IP adresli URL, aşırı uzun URL, garip karakterli URL, içinde "login" geçen ama normal olan URL (false positive kontrolü)

*Arayüz*
- [ ] API katmanı (FastAPI)
- [ ] Basit, tek sayfalık web arayüzü

**Faz 2 — Genişleme**
- [ ] Telegram botu (aynı API'yi kullanır, yeni bir giriş kapısı)

**Faz 3 — Opsiyonel / "gelecek çalışmalar"**
- [ ] PWA desteği (siteyi uygulama gibi telefona kurma)
- [ ] Android paylaş menüsü entegrasyonu

## Teknoloji
- **Python** — analiz motoru
- **FastAPI** — API katmanı
- **HTML/basit frontend** — arayüz
- **Git/GitHub** — versiyon kontrolü
- **(Faz 2) Telegram Bot API**

## Öğrenme Hedefleri
- **Git:** add, commit, push, branch, merge, diff, log, geri alma — hepsi gerçek geliştirme sürecinde, feature branch akışıyla uygulamalı olarak
- **Python:** fonksiyon tasarımı, regex, modüler kod yapısı
- **Backend:** API tasarımı, istek/cevap mantığı
- **Siber güvenlik:** phishing göstergeleri, URL analizi, sosyal mühendislik dil paternleri

## Neden bu şekilde yapıyoruz (kısa hatırlatma)
Tespit mantığı bir LLM'e sorularak değil, **kendi yazdığımız kurallarla** çalışıyor — hem gerçek siber güvenlik bilgisi kazandırıyor hem de dışarıdan gelen (güvenilmeyen) metni doğrudan bir LLM'e vermenin getirdiği prompt injection riskini taşımıyor.