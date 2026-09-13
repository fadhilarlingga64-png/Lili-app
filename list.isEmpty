import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:dio/dio.dart';
import 'package:gal/gal.dart';
import 'package:path_provider/path_provider.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:cached_network_image/cached_network_image.dart';

const String API = "https://tolss-api.up.railway.app";

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const TolssApp());
}

class TolssApp extends StatelessWidget {
  const TolssApp({super.key});
  @override
  Widget build(BuildContext context) => MaterialApp(
    title: 'TOLSS',
    debugShowCheckedModeBanner: false,
    theme: ThemeData(
      useMaterial3: true,
      brightness: Brightness.dark,
      colorScheme: ColorScheme.fromSeed(
        seedColor: const Color(0xFF6C5CE7),
        brightness: Brightness.dark,
      ),
      scaffoldBackgroundColor: const Color(0xFF0F0F1E),
    ),
    home: const HomePage(),
  );
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final urlCtrl = TextEditingController();
  bool loading = false;
  Map? result;
  String tab = "Video";

  Future<void> _paste() async {
    final d = await Clipboard.getData('text/plain');
    if (d?.text != null) urlCtrl.text = d!.text!;
  }

  Future<void> _fetch() async {
    if (urlCtrl.text.trim().isEmpty) return _snack("Tempel link dulu");
    setState(() { loading = true; result = null; });
    try {
      final r = await Dio().get("$API/info", queryParameters: {"url": urlCtrl.text.trim()});
      if (r.data['error'] != null) throw r.data['error'];
      setState(() => result = r.data);
    } catch (e) {
      _snack("❌ $e");
    } finally {
      setState(() => loading = false);
    }
  }

  Future<void> _download(String url, String type) async {
    if (type == 'video') await Permission.videos.request();
    else await Permission.photos.request();
    try {
      final dir = await getTemporaryDirectory();
      final ext = type == 'image' ? 'jpg' : 'mp4';
      final file = "${dir.path}/TOLSS_${DateTime.now().millisecondsSinceEpoch}.$ext";
      _snack("⏳ Mengunduh...");
      await Dio().download("$API/download", file, queryParameters: {"url": url});
      if (type == 'image') await Gal.putImage(file);
      else await Gal.putVideo(file);
      _snack("✅ Tersimpan di Galeri!");
    } catch (e) {
      _snack("❌ $e");
    }
  }

  void _snack(String m) => ScaffoldMessenger.of(context)
    ..hideCurrentSnackBar()
    ..showSnackBar(SnackBar(content: Text(m), behavior: SnackBarBehavior.floating));

  @override
  Widget build(BuildContext context) {
    final videos = (result?['formats'] as List? ?? []).where((f) => f['type'] == 'video').toList();
    final images = (result?['formats'] as List? ?? []).where((f) => f['type'] == 'image').toList();
    final list = tab == "Video" ? videos : images;

    return Scaffold(
      body: SafeArea(
        child: Column(children: [
          const Padding(
            padding: EdgeInsets.all(20),
            child: Row(children: [
              Icon(Icons.download_rounded, color: Color(0xFF6C5CE7), size: 32),
              SizedBox(width: 10),
              Text("TOLSS", style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold, letterSpacing: 2)),
            ]),
          ),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: TextField(
              controller: urlCtrl,
              style: const TextStyle(fontSize: 14),
              decoration: InputDecoration(
                hintText: "Tempel link di sini...",
                filled: true,
                fillColor: const Color(0xFF1A1A2E),
                border: OutlineInputBorder(borderRadius: BorderRadius.circular(14), borderSide: BorderSide.none),
                suffixIcon: IconButton(icon: const Icon(Icons.paste), onPressed: _paste),
              ),
            ),
          ),
          const SizedBox(height: 12),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: SizedBox(
              width: double.infinity,
              height: 50,
              child: FilledButton(
                onPressed: loading ? null : _fetch,
                style: FilledButton.styleFrom(
                  backgroundColor: const Color(0xFF6C5CE7),
                  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(14)),
                ),
                child: Text(loading ? "Loading..." : "Ambil Media",
                    style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 15)),
              ),
            ),
          ),
          const SizedBox(height: 12),
          if (result != null) ...[
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Row(children: [
                Expanded(child: _tabBtn("Video", videos.length)),
                const SizedBox(width: 8),
                Expanded(child: _tabBtn("Foto", images.length)),
              ]),
            ),
            const SizedBox(height: 8),
          ],
          Expanded(
            child: list.isEmpty
                ? Center(child: Text(
                    result == null ? "Support 1000+ situs 🌐\nTikTok • IG • YT • FB • X" : "Tidak ada media",
                    textAlign: TextAlign.center,
                    style: TextStyle(color: Colors.grey[600])))
                : ListView.builder(
                    padding: const EdgeInsets.symmetric(horizontal: 16),
                    itemCount: list.length,
                    itemBuilder: (_, i) {
                      final f = list[i];
                      return Card(
                        color: const Color(0xFF1A1A2E),
                        margin: const EdgeInsets.only(bottom: 8),
                        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                        child: ListTile(
                          title: Text("${f['quality']} • ${f['ext']}",
                              style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 13)),
                          subtitle: Text(
                            f['watermark'] == true ? "⚠️ Watermark" : "✨ No watermark",
                            style: TextStyle(
                                fontSize: 11,
                                color: f['watermark'] == true ? Colors.orange : Colors.green),
                          ),
                          trailing: IconButton(
                            icon: const Icon(Icons.download, color: Color(0xFF6C5CE7)),
                            onPressed: () => _download(f['url'], f['type']),
                          ),
                        ),
                      );
                    },
                  ),
          ),
        ]),
      ),
    );
  }

  Widget _tabBtn(String label, int n) {
    final active = tab == label;
    return GestureDetector(
      onTap: () => setState(() => tab = label),
      child: Container(
        padding: const EdgeInsets.symmetric(vertical: 10),
        decoration: BoxDecoration(
          color: active ? const Color(0xFF6C5CE7) : const Color(0xFF1A1A2E),
          borderRadius: BorderRadius.circular(12),
        ),
        child: Center(child: Text("$label ($n)",
            style: TextStyle(fontWeight: FontWeight.bold, color: active ? Colors.white : Colors.grey))),
      ),
    );
  }
}
