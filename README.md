# -
برنامج يقوم بتحديد أوقات الصلاة و الثلث الأخير من الليل و الثالث الثالث من الليل الذي يتنزل فيه ربنا جلَّ في علاه
import 'package:flutter/material.dart';
import 'dart:async';
import 'package:sensors_plus/sensors_plus.dart';
import 'package:torch_light/torch_light.dart';
import 'package:share_plus/share_plus.dart';

void main() => runApp(const ThuluthApp());

class ThuluthApp extends StatelessWidget {
  const ThuluthApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData.dark().copyWith(
        scaffoldBackgroundColor: const Color(0xFF0B0D17),
        primaryColor: const Color(0xFFFFD700),
      ),
      home: const HomeScreen(),
    );
  }
}

class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});
  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  bool isFlashing = false;
  String selectedCity = "باتنة، الجزائر";

  @override
  void initState() {
    super.initState();
    // تفعيل حساس قلب الهاتف للإسكات
    accelerometerEvents.listen((AccelerometerEvent event) {
      if (event.z < -8.0 && isFlashing) {
        stopAlert();
      }
    });
  }

  void startAlert() {
    setState(() => isFlashing = true);
    // منطق الوميض المتقطع
    Timer.periodic(const Duration(milliseconds: 500), (timer) {
      if (!isFlashing) {
        timer.cancel();
        TorchLight.disableTorch();
      } else {
        (timer.tick % 2 == 0) ? TorchLight.enableTorch() : TorchLight.disableTorch();
      }
    });
  }

  void stopAlert() {
    setState(() => isFlashing = false);
    TorchLight.disableTorch();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("الثلث الأخير", style: TextStyle(color: Color(0xFFFFD700))),
        backgroundColor: Colors.transparent,
        elevation: 0,
        actions: [
          IconButton(icon: const Icon(Icons.share), onPressed: () => Share.share("تذكير بالثلث الأخير في $selectedCity")),
        ],
      ),
      body: SingleChildScrollView(
        child: Column(
          children: [
            const SizedBox(height: 20),
            Text(selectedCity, style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
            const SizedBox(height: 30),
            _buildPrayerCard("المغرب", "18:45", Icons.wb_sunny_outlined),
            _buildPrayerCard("الفجر", "05:30", Icons.wb_twilight),
            const Divider(color: Color(0xFFFFD700), indent: 50, endIndent: 50),
            _buildThirdCard("الجزء الأول من الثلث", "01:30 AM"),
            _buildThirdCard("الجزء الثاني من الثلث", "02:20 AM"),
            _buildThirdCard("الجزء الثالث من الثلث", "03:10 AM"),
            const SizedBox(height: 40),
            ElevatedButton.icon(
              onPressed: startAlert,
              icon: const Icon(Icons.play_arrow),
              label: const Text("تجربة التنبيه والوميض"),
              style: ElevatedButton.styleFrom(backgroundColor: Colors.amber[800]),
            )
          ],
        ),
      ),
    );
  }

  Widget _buildPrayerCard(String label, String time, IconData icon) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 20, vertical: 5),
      child: ListTile(
        leading: Icon(icon, color: Colors.blueAccent),
        title: Text(label),
        trailing: Text(time, style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
      ),
    );
  }

  Widget _buildThirdCard(String label, String time) {
    return Card(
      color: const Color(0xFF1C2031),
      margin: const EdgeInsets.symmetric(horizontal: 20, vertical: 5),
      child: ListTile(
        leading: const Icon(Icons.notifications_active, color: Color(0xFFFFD700)),
        title: Text(label),
        subtitle: const Text("تنبيه + وميض"),
        trailing: Text(time, style: const TextStyle(color: Color(0xFFFFD700))),
      ),
    );
  }
}
