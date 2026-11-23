# Token tasarrufu kuralları

## Çalışma kuralları (Claude için)
- Önce CLAUDE.md'deki yapıya güven; kod tabanını keşfetme. Yalnız ilgili dosyayı oku, gerekirse `grep -n` ile satır bul.
- Büyük dosyaları (`koleksiyonlar.json`, `app.css`, `package-lock.json`) tamamen okuma; `offset/limit` veya `grep` kullan.
- Bağımsız komutları tek mesajda paralel çalıştır; gereksiz ara kontrol, tekrar okuma, "doğrulama" turu yapma.
- Tarayıcı testi, ekran görüntüsü, uzun log: yalnız istenirse. Kontrol = `npx tsc -p tsconfig.json`.
- Alt ajan (Agent), plan modu, beyin fırtınası/skill zincirleri: yalnız istenirse. Basit işi doğrudan yap.
- Yanıt kısa: ne yapıldı + link. Özet tekrarı, seçenek listesi, uzun açıklama yok.
- Aynı bilgiyi tekrar türetme; konuşmada zaten kesinleşmiş kararı yeniden tartışma.
- Yayın: `tsc` geçince `main`'e commit + push; Actions kendiliğinden yayınlar.

## Neden çok token harcanıyor?
1. **Her mesajda sabit yük:** Claude'un kurulu eklentileri (yüzlerce skill, ajan, MCP aracı: Adobe, Vercel, Shopify, finans, vb.) her turda bağlama yüklenir. Bu, sen tek kelime yazsan bile on binlerce token demek.
2. **Konuşma uzadıkça büyür:** Her yeni mesajda önceki tüm konuşma + araç çıktıları yeniden gönderilir (önbellekle ucuzlasa da sayılır).
3. **Büyük araç çıktıları:** Uzun dosya/log okumak, `git pull` farkları, tarama çıktıları bağlama girer.
4. **Skill/ajan tetikleme:** Bazı eklentiler "her işte önce skill çağır" der; bu ek okuma ve adım demek.
5. **Gereksiz doğrulama turları:** Ek kontrol, tekrar okuma, ekran görüntüsü.

## Sen ne yapabilirsin (en büyük kazanç)
- Kullanmadığın eklentileri/MCP sunucularını kapat (Ayarlar → Eklentiler / Bağlayıcılar). En büyük sabit yükü bu azaltır.
- Konu değişince yeni oturum aç; uzun oturumda `/compact` veya `/clear` kullan.
- Küçük işleri tek mesajda, net iste ("X dosyasında Y'yi değiştir").
- Daha hafif model (Haiku/Sonnet) basit işler için yeterli.
