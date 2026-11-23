# Su's Magical Journal

Kişisel yaşam + sanat kariyeri + proje yönetimi uygulaması. Arayüz tamamen **Türkçe**, saat dilimi **Europe/Istanbul**.

## Yığın
- React 18 + TypeScript + Vite, TanStack Query, react-router (basename = `import.meta.env.BASE_URL`)
- Supabase projesi `tjouaydmwmokbuomyyub` (Postgres + RLS, Storage kovası `files`, Realtime, Edge Functions, pg_cron)
- Node: `export PATH="$HOME/.local/node/bin:$HOME/.local/bin:$PATH"` (gh ve supabase CLI de burada)
- Yayın: `main`'e push → GitHub Actions → https://busrasuhaydar.github.io/su-magical-journal/ (`GH_PAGES=1` ile base `/su-magical-journal/`)

## Komutlar
- Geliştirme: `npm run dev` (port 5173, strictPort)
- Kontrol: `npx tsc -p tsconfig.json` · Derleme: `npm run build`
- DB göçü: `supabase/migrations/` içine dosya yaz. `supabase db push` bu ağda çalışmıyor (IPv6); kısa DDL için MCP `apply_migration`, veri için `execute_sql` kullan (uzun SQL yapıştırma)
- Fonksiyon: `supabase functions deploy <ad> --project-ref tjouaydmwmokbuomyyub` (dispatcher ve oauth için `--no-verify-jwt`)

## Yapı
- `src/lib/data.ts` — tüm sorgular/mutasyonlar; `TABLE_KEYS` Realtime → sorgu yenileme eşlemesi (yeni tablo eklenirse buraya VE `supabase_realtime` yayınına ekle, yoksa tüm Realtime kanalı bozulur)
- `src/lib/dates.ts` — gün anahtarı `YYYY-MM-DD`, `toInstant/dayKeyOf/minutesOf` (İstanbul)
- `src/lib/types.ts` — tipler ve Türkçe etiketler
- `src/lib/nft.ts` — NFT verisi (`public/koleksiyonlar.json`: koleksiyonlar + 530 eser, orijinaller IPFS/Arweave/OpenSea'de; kopya tutulmaz), koleksiyon→proje eşitleme (`koleksiyon-<slug>`), orijinal kaydet/paylaş (`withWorkName`: paylaşım metninin başına eser adı)
- `src/lib/ui.ts` — ana sayfa özelleştirmesi (`profiles.ui_prefs`: kart başlıkları, gizleme, Boston saati); `src/lib/prefs.ts` — cihaza özel tercihler (şehir, sade mod); `src/lib/autoIcons.tsx` — simgesiz `.btn`'lere otomatik simge
- `src/components/` — Shell (menü, hava durumu `useWeather`), Editors (görev/etkinlik/içerik pencereleri, `useEditors`), TimeGrid, PostEditor, QuickSearch (⌘K; eserler dahil), NotificationBell, NftLightbox (galeri penceresi + "Bu eserle çalış": sayfa/fikir/görev)
- `src/pages/` — Home (hero, plan, Hatırlatmalar/düzenli işler, yapılacaklar, günün eseri, bu hafta), Calendar (ay/hafta/ajanda: görev, etkinlik, takip, kilometre taşı, proje hedefi, tekrar projeksiyonu), Day, Tasks, Projects, project/{Workspace,Gallery,Pages,Links,Sections}, Social, Opportunities, Contacts, Ideas, Gallery (ortak; "NFT eserleri" sekmesi), NftCollections (`/koleksiyonlar`, `/koleksiyonlar/:slug`, Eserler + Sayfalar sekmeleri), Files, Settings
- `supabase/functions/` — `extract-opportunity` (Claude ile duyuru çıkarma), `dispatcher` (her dakika: hatırlatıcı→bildirim/web push, yayın kuyruğu), `oauth`
- Stil: `src/styles/app.css` (en alttaki "Liquid Glass v2" ve sonraki bloklar belirteçleri geçersiz kılar; yeni stil hep dosyanın SONUNA)

## Kurallar
- Tek kayıt ilkesi: veri kopyalanmaz; ilişkiler `file_links`, `entity_links`, `project_*` tablolarıyla.
- Her tabloda `owner_id default auth.uid()` + RLS; değişiklikler `change_log`'a tetikleyiciyle yazılır; silme = `deleted_at` (yumuşak).
- `memory_entries` değiştirilemez (yalnız ekleme, `supersedes`).
- Sosyal paylaşım şu an elle: `src/lib/share.ts` (paylaş menüsü / X intent); paylaşımlar `post_shares`'e kaydedilir. API ile otomatik yayın altyapısı hazır ama anahtarlar girilmedi.
- Belirsiz bilgi uydurulmaz; dinî bayramlar Diyanet takvimi yayımlanana kadar `certainty='uncertain'` (2026–2027 kesinleşti, 2028 değil). Takvimde yalnız TR ve US tatilleri tutulur/gösterilir.
- Tekrarlayan görev (`rrule`): `complete_task` sonraki örneği ve hatırlatıcılarını aynı aralıkla taşır. Koleksiyon = proje; eser sayfaları `pages.artwork_key = '<sözleşme>-<tokenId>'` ile bağlanır.
- Her şey birbiriyle ilişkili olmalı (canlı organizma): yeni özellik mevcut kayıtlara (proje, sayfa, fikir, görev, takvim, arama ⌘K) bağlanmalı, ayrı ada olmamalı.
- Kullanıcıya Türkçe, kısa yanıt ver; token tasarrufu önemli.

## Arayüz kuralları (kullanıcı kesin istedi)
- Her butonun simgesi olur (lucide-react); metin oku (`←`) yerine simge. Kopyala yalnız simge; OpenSea/SuperRare link düğmeleri "OS"/"SR".
- Hiçbir öğe hiçbir cihazda çakışmaz: sabit konumlu öğe ekleme (arama/bildirim `.utilbar` ile akış içinde). Yeni sayfada 375px ve masaüstünde taşma/çakışma kontrol et.
- Arayüz Türkçe; ana sayfa Bursa hava durumu + küçük Boston saati.

## Token tasarrufu (önemli)
- İstenmedikçe kod tabanını keşfetme; CLAUDE.md'deki yapıya güven, yalnız ilgili dosyayı oku (gerekirse `grep -n` ile ilgili satırları).
- İstenmedikçe tarayıcı testi, ekran görüntüsü, uzun log okuma yapma. Kontrol = `npx tsc -p tsconfig.json`.
- Yayın: değişikliği `main`'e commit + push (Actions kendiliğinden yayınlar). Canlıda eski sürüm görünürse service worker önbelleği: sert yenile ya da `?v=<sayı>` ekle.
- Komut zincirinde `sed -i` macOS'ta bozuk; düzenleme için Python/Edit kullan ve `tsc` geçmeden commit/push etme.
- Yanıtlar kısa olsun.

- Token kuralları ve nedenleri: `TOKEN-TASARRUFU.md` (uygula).
