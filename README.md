# Axis
A single, customizable home screen that aggregates the user’s most important information in real time.
npx create-expo-app axis -t expo-router
cd axis
npm install firebase zustand axios
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID",
};

const app = initializeApp(firebaseConfig);

export const auth = getAuth(app);
export const db = getFirestore(app);
import { ScrollView } from "react-native";
import CryptoWidget from "@/components/widgets/CryptoWidget";
import WeatherWidget from "@/components/widgets/WeatherWidget";

export default function Dashboard() {
  return (
    <ScrollView style={{ padding: 16 }}>
      <CryptoWidget />
      <WeatherWidget />
    </ScrollView>
  );
}
import { View, StyleSheet } from "react-native";

export default function WidgetCard({ children }: { children: React.ReactNode }) {
  return <View style={styles.card}>{children}</View>;
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: "#111827",
    borderRadius: 16,
    padding: 16,
    marginBottom: 16,
  },
});
import { Text } from "react-native";
import WidgetCard from "../ui/WidgetCard";

export default function CryptoWidget() {
  return (
    <WidgetCard>
      <Text style={{ color: "white", fontSize: 18 }}>Bitcoin</Text>
      <Text style={{ color: "#10b981" }}>$43,210</Text>
    </WidgetCard>
  );
}
npm install expo-location
import { create } from "zustand";

interface DashboardState {
  cryptoPrice: number | null;
  weather: {
    temp: number;
    condition: string;
  } | null;

  setCryptoPrice: (price: number) => void;
  setWeather: (temp: number, condition: string) => void;
}

export const useDashboardStore = create<DashboardState>((set) => ({
  cryptoPrice: null,
  weather: null,

  setCryptoPrice: (price) => set({ cryptoPrice: price }),
  setWeather: (temp, condition) =>
    set({ weather: { temp, condition } }),
}));
import axios from "axios";

export async function fetchBitcoinPrice() {
  const res = await axios.get(
    "https://api.coingecko.com/api/v3/simple/price",
    {
      params: {
        ids: "bitcoin",
        vs_currencies: "usd",
      },
    }
  );

  return res.data.bitcoin.usd;
}
import axios from "axios";

const API_KEY = "YOUR_OPENWEATHER_KEY";

export async function fetchWeather(lat: number, lon: number) {
  const res = await axios.get(
    "https://api.openweathermap.org/data/2.5/weather",
    {
      params: {
        lat,
        lon,
        units: "metric",
        appid: API_KEY,
      },
    }
  );

  return {
    temp: res.data.main.temp,
    condition: res.data.weather[0].main,
  };
}
import { Text } from "react-native";
import { useEffect } from "react";
import WidgetCard from "../ui/WidgetCard";
import { fetchBitcoinPrice } from "@/services/crypto";
import { useDashboardStore } from "@/state/dashboardStore";

export default function CryptoWidget() {
  const price = useDashboardStore((s) => s.cryptoPrice);
  const setPrice = useDashboardStore((s) => s.setCryptoPrice);

  useEffect(() => {
    fetchBitcoinPrice().then(setPrice);
  }, []);

  return (
    <WidgetCard>
      <Text style={{ color: "white", fontSize: 18 }}>Bitcoin</Text>
      <Text style={{ color: "#10b981", fontSize: 22 }}>
        {price ? `$${price}` : "Loading..."}
      </Text>
    </WidgetCard>
  );
}
import { Text } from "react-native";
import { useEffect } from "react";
import * as Location from "expo-location";
import WidgetCard from "../ui/WidgetCard";
import { fetchWeather } from "@/services/weather";
import { useDashboardStore } from "@/state/dashboardStore";

export default function WeatherWidget() {
  const weather = useDashboardStore((s) => s.weather);
  const setWeather = useDashboardStore((s) => s.setWeather);

  useEffect(() => {
    (async () => {
      const { status } =
        await Location.requestForegroundPermissionsAsync();
      if (status !== "granted") return;

      const loc = await Location.getCurrentPositionAsync({});
      const data = await fetchWeather(
        loc.coords.latitude,
        loc.coords.longitude
      );

      setWeather(data.temp, data.condition);
    })();
  }, []);

  return (
    <WidgetCard>
      <Text style={{ color: "white", fontSize: 18 }}>Weather</Text>
      <Text style={{ color: "white" }}>
        {weather
          ? `${weather.temp}°C · ${weather.condition}`
          : "Loading..."}
      </Text>
    </WidgetCard>
  );
}
import { View, Text, Button } from "react-native";
import { signInAnonymously } from "firebase/auth";
import { auth } from "@/services/firebase";

export default function Login() {
  return (
    <View style={{ padding: 24 }}>
      <Text style={{ fontSize: 24 }}>Axis</Text>
      <Button
        title="Continue"
        onPress={() => signInAnonymously(auth)}
      />
    </View>
  );
}
import { Stack } from "expo-router";
import { onAuthStateChanged } from "firebase/auth";
import { auth } from "@/services/firebase";
import { useEffect, useState } from "react";

export default function Layout() {
  const [user, setUser] = useState<any>(null);

  useEffect(() => {
    return onAuthStateChanged(auth, setUser);
  }, []);

  return (
    <Stack>
      {!user ? (
        <Stack.Screen name="(auth)/login" />
      ) : (
        <Stack.Screen name="(dashboard)" />
      )}
    </Stack>
  );
}
import { Stack } from "expo-router";
import { onAuthStateChanged } from "firebase/auth";
import { auth } from "@/services/firebase";
import { useEffect, useState } from "react";

export default function Layout() {
  const [user, setUser] = useState<any>(null);

  useEffect(() => {
    return onAuthStateChanged(auth, setUser);
  }, []);

  return (
    <Stack>
      {!user ? (
        <Stack.Screen name="(auth)/login" />
      ) : (
        <Stack.Screen name="(dashboard)" />
      )}
    </Stack>
  );
}
