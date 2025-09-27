
import streamlit as st
import folium
from folium.plugins import MarkerCluster
from streamlit_folium import st_folium
import re

st.title("পূজা প্যান্ডেল রুট অপ্টিমাইজার")

uploaded_file = st.file_uploader("আপনার KML ফাইল আপলোড করুন", type=["txt", "kml"])

if uploaded_file is not None:
    content = uploaded_file.read().decode("utf-8")
    coordinates = re.findall(r"([\\d\\.\\-]+),([\\d\\.\\-]+)", content)
    coords = [(float(lat), float(lon)) for lon, lat in coordinates]

    if coords:
        st.success(f"মোট {len(coords)} টি লোকেশন পাওয়া গেছে।")

        def nearest_neighbor_route(points):
            route = [points[0]]
            remaining = points[1:]
            while remaining:
                last = route[-1]
                next_point = min(remaining, key=lambda p: (p[0]-last[0])**2 + (p[1]-last[1])**2)
                route.append(next_point)
                remaining.remove(next_point)
            return route

        optimized_route = nearest_neighbor_route(coords)

        m = folium.Map(location=optimized_route[0], zoom_start=13)
        marker_cluster = MarkerCluster().add_to(m)

        for idx, point in enumerate(optimized_route):
            folium.Marker(location=point, popup=f"স্টপ {idx+1}").add_to(marker_cluster)

        folium.PolyLine(optimized_route, color="blue", weight=3, opacity=0.8).add_to(m)

        st_folium(m, width=700, height=500)
    else:
        st.error("কোনো কোঅর্ডিনেট পাওয়া যায়নি। অনুগ্রহ করে ফাইলটি যাচাই করুন।")
else:
    st.info("শুরু করতে একটি KML ফাইল আপলোড করুন।")
