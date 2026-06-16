import customtkinter as ctk
from tkinter import filedialog, messagebox
import pygame
import os
import tkinter as tk
from mutagen.mp3 import MP3

# Set tema gelap pekat ala Spotify
ctk.set_appearance_mode("Dark")

class PremiumSpotifyClone(ctk.CTk):
    def __init__(self):
        super().__init__()

        self.title("Spotify")
        self.geometry("950x650")
        self.configure(fg_color="#000000") # Black background

        # Inisialisasi Audio
        pygame.mixer.init()

        # State Aplikasi
        self.playlist = []
        self.current_song_index = None
        self.is_paused = False
        self.song_length = 0
        self.current_time = 0
        self.is_dragging = False
        self.default_folder = "musik"

        # --- 1. SIDEBAR (KIRI) ---
        self.sidebar = ctk.CTkFrame(self, width=220, corner_radius=0, fg_color="#000000")
        self.sidebar.pack(side=tk.LEFT, fill=tk.Y)
        self.sidebar.pack_propagate(False)

        # Isi Sidebar
        logo = ctk.CTkLabel(self.sidebar, text="Spotify", font=("Arial", 24, "bold"), text_color="#ffffff", anchor="w")
        logo.pack(pady=(20, 20), padx=25, fill=tk.X)

        self.create_sidebar_btn("🏠  Home", active=True)
        self.create_sidebar_btn("🔍  Search", active=False)
        self.create_sidebar_btn("📚  Your Library", active=False)
        
        # --- 2. PLAYER BAR (BAWAH) ---
        self.player_bar = ctk.CTkFrame(self, height=90, corner_radius=0, fg_color="#000000", border_width=1, border_color="#282828")
        self.player_bar.pack(side=tk.BOTTOM, fill=tk.X)
        self.player_bar.pack_propagate(False)

        # --- 3. MAIN CONTENT (TENGAH) ---
        self.main_content = ctk.CTkScrollableFrame(self, corner_radius=8, fg_color="#121212")
        self.main_content.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=(0, 10), pady=(10, 0))

        # --- ISI MAIN CONTENT ---
        # Banner Atas
        self.banner = ctk.CTkFrame(self.main_content, height=120, fg_color="transparent")
        self.banner.pack(fill=tk.X, padx=20, pady=10)
        
        banner_lbl = ctk.CTkLabel(self.banner, text="PLAYLIST", font=("Arial", 10, "bold"), text_color="#ffffff")
        banner_lbl.pack(anchor="w", pady=(20, 0))
        
        self.playlist_title = ctk.CTkLabel(self.banner, text="Koleksi Lagu Lokal", font=("Arial", 36, "bold"), text_color="#ffffff")
        self.playlist_title.pack(anchor="w")

        # Baris Tombol Aksi (Play Besar Hijau)
        action_bar = ctk.CTkFrame(self.main_content, fg_color="transparent", height=60)
        action_bar.pack(fill=tk.X, padx=20, pady=10)
        
        self.big_play_btn = ctk.CTkButton(action_bar, text="▶️", width=50, height=50, corner_radius=25, fg_color="#1DB954", text_color="black", font=("Arial", 20, "bold"), hover_color="#1ed760", command=self.toggle_play)
        self.big_play_btn.pack(side=tk.LEFT)

        folder_btn = ctk.CTkButton(action_bar, text="📁 Ganti Folder", width=100, fg_color="transparent", text_color="#b3b3b3", hover_color="#282828", font=("Arial", 12, "bold"), command=self.change_folder)
        folder_btn.pack(side=tk.LEFT, padx=20)

        # Header Tabel Lagu
        table_header = ctk.CTkFrame(self.main_content, fg_color="transparent", height=25)
        table_header.pack(fill=tk.X, padx=20, pady=(10, 5))
        
        ctk.CTkLabel(table_header, text="#", text_color="#b3b3b3", width=30, anchor="w", font=("Arial", 11, "bold")).pack(side=tk.LEFT)
        ctk.CTkLabel(table_header, text="JUDUL", text_color="#b3b3b3", anchor="w", font=("Arial", 11, "bold")).pack(side=tk.LEFT, padx=10)
        ctk.CTkLabel(table_header, text="🕒", text_color="#b3b3b3", anchor="e", font=("Arial", 11, "bold")).pack(side=tk.RIGHT, padx=25)

        # Garis Pembatas
        devider = ctk.CTkFrame(self.main_content, height=1, fg_color="#282828")
        devider.pack(fill=tk.X, padx=20, pady=5)

        # Container untuk List Lagu
        self.tracks_container = ctk.CTkFrame(self.main_content, fg_color="transparent")
        self.tracks_container.pack(fill=tk.BOTH, expand=True, padx=10)

        # --- ISI PLAYER BAR (BAWAH) ---
        # Kiri: Info Lagu
        self.info_area = ctk.CTkFrame(self.player_bar, fg_color="transparent", width=220)
        self.info_area.pack(side=tk.LEFT, fill=tk.Y, padx=15)
        self.info_area.pack_propagate(False)

        self.track_name_lbl = ctk.CTkLabel(self.info_area, text="Tidak ada lagu", font=("Arial", 13, "bold"), text_color="#ffffff", anchor="w")
        self.track_name_lbl.pack(pady=(25, 0), fill=tk.X)
        self.artist_lbl = ctk.CTkLabel(self.info_area, text="Lokal File", font=("Arial", 11), text_color="#b3b3b3", anchor="w")
        self.artist_lbl.pack(fill=tk.X)

        # Tengah: Kontrol Utama & Slider
        self.controls_area = ctk.CTkFrame(self.player_bar, fg_color="transparent")
        self.controls_area.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

        # Tombol Media Atas
        mid_btns = ctk.CTkFrame(self.controls_area, fg_color="transparent")
        mid_btns.pack(pady=(10, 2))

        self.prev_btn = ctk.CTkButton(mid_btns, text="⏮", width=35, fg_color="transparent", text_color="#b3b3b3", font=("Arial", 16), hover_color="#181818", command=self.prev_song)
        self.prev_btn.grid(row=0, column=0, padx=10)

        self.core_play_btn = ctk.CTkButton(mid_btns, text="▶️", width=35, height=35, corner_radius=18, fg_color="#ffffff", text_color="black", font=("Arial", 12, "bold"), hover_color="#e6e6e6", command=self.toggle_play)
        self.core_play_btn.grid(row=0, column=1, padx=10)

        self.next_btn = ctk.CTkButton(mid_btns, text="⏭", width=35, fg_color="transparent", text_color="#b3b3b3", font=("Arial", 16), hover_color="#181818", command=self.next_song)
        self.next_btn.grid(row=0, column=2, padx=10)

        # Slider Bawah
        slider_row = ctk.CTkFrame(self.controls_area, fg_color="transparent")
        slider_row.pack(fill=tk.X, padx=40)

        self.time_passed = ctk.CTkLabel(slider_row, text="0:00", font=("Arial", 11), text_color="#b3b3b3", width=35)
        self.time_passed.pack(side=tk.LEFT)

        self.seek_slider = ctk.CTkSlider(slider_row, from_=0, to=100, number_of_steps=1000, fg_color="#404040", progress_color="#1DB954", button_color="#ffffff", button_hover_color="#1ed760", height=12)
        self.seek_slider.pack(side=tk.LEFT, fill=tk.X, expand=True, padx=5)
        self.seek_slider.set(0)
        self.seek_slider.bind("<ButtonPress-1>", self.on_slider_press)
        self.seek_slider.bind("<ButtonRelease-1>", self.on_slider_release)

        self.time_total = ctk.CTkLabel(slider_row, text="0:00", font=("Arial", 11), text_color="#b3b3b3", width=35)
        self.time_total.pack(side=tk.LEFT)

        # Kanan: Fitur Tambahan (Dummy / Volume)
        right_area = ctk.CTkFrame(self.player_bar, fg_color="transparent", width=180)
        right_area.pack(side=tk.RIGHT, fill=tk.Y, padx=15)
        
        vol_lbl = ctk.CTkLabel(right_area, text="🔊", text_color="#b3b3b3", font=("Arial", 12))
        vol_lbl.pack(side=tk.LEFT, padx=(20, 5), pady=(15, 0))
        
        self.vol_slider = ctk.CTkSlider(right_area, from_=0, to=1, width=90, fg_color="#404040", progress_color="#ffffff", button_color="#ffffff", height=10, command=lambda v: pygame.mixer.music.set_volume(v))
        self.vol_slider.pack(side=tk.LEFT, pady=(23, 0))
        self.vol_slider.set(0.7)

        # Auto Load Folder Musik
        self.auto_load_songs()

    def create_sidebar_btn(self, text, active=False):
        color = "#ffffff" if active else "#b3b3b3"
        font_weight = "bold" if active else "normal"
        btn = ctk.CTkButton(self.sidebar, text=text, fg_color="transparent", text_color=color, anchor="w", font=("Arial", 14, font_weight), hover_color="#121212", height=40)
        btn.pack(fill=tk.X, padx=15, pady=2)

    # --- LOGIKA APLIKASI ---
    def auto_load_songs(self):
        current_dir = os.path.dirname(os.path.abspath(__file__))
        self.default_folder = os.path.join(current_dir, "musik")
        if not os.path.exists(self.default_folder):
            try: os.makedirs(self.default_folder)
            except: pass
        self.after(100, lambda: self.load_from_directory(self.default_folder))

    def load_from_directory(self, path):
        self.playlist = []
        try:
            for file in os.listdir(path):
                if file.endswith(".mp3"):
                    self.playlist.append(os.path.join(path, file))
            self.update_songs_table_ui()
        except Exception as e:
            messagebox.showerror("Error", f"Gagal memuat folder: {e}")

    def change_folder(self):
        selected = filedialog.askdirectory()
        if selected: self.load_from_directory(selected)

    def format_time(self, seconds):
        return f"{int(seconds // 60)}:{int(seconds % 60):02d}"

    def update_songs_table_ui(self):
        for widget in self.tracks_container.winfo_children(): widget.destroy()
        
        if not self.playlist:
            ctk.CTkLabel(self.tracks_container, text="Folder 'musik' kosong. Taruh beberapa file .mp3 di sana!", text_color="#b3b3b3").pack(pady=30)
            return

        for index, path in enumerate(self.playlist):
            song_name = os.path.basename(path)
            
            # Membaca durasi lagu asli untuk ditaruh di kolom kanan tabel
            try:
                audio = MP3(path)
                duration_str = self.format_time(audio.info.length)
            except:
                duration_str = "--:--"

            row = ctk.CTkFrame(self.tracks_container, fg_color="transparent", height=45, corner_radius=4)
            row.pack(fill=tk.X, pady=2)
            row.pack_propagate(False)

            # Efek hover highlight baris ala Spotify
            row.bind("<Enter>", lambda e, r=row: r.configure(fg_color="#2a2a2a"))
            row.bind("<Leave>", lambda e, r=row: r.configure(fg_color="transparent"))

            # Kolom No
            num_lbl = ctk.CTkLabel(row, text=str(index + 1), text_color="#b3b3b3", width=30, anchor="w")
            num_lbl.pack(side=tk.LEFT, padx=10)

            # Kolom Judul (Berfungsi sebagai tombol klik)
            title_btn = ctk.CTkButton(row, text=song_name, anchor="w", fg_color="transparent", text_color="#ffffff", font=("Arial", 13), hover_color="#2a2a2a", command=lambda i=index: self.play_song(i))
            title_btn.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

            # Kolom Durasi
            time_lbl = ctk.CTkLabel(row, text=duration_str, text_color="#b3b3b3", width=50, anchor="e")
            time_lbl.pack(side=tk.RIGHT, padx=20)

    def play_song(self, index):
        self.current_song_index = index
        song_path = self.playlist[index]
        
        audio = MP3(song_path)
        self.song_length = audio.info.length
        
        self.current_time = 0
        self.seek_slider.configure(to=self.song_length)
        self.seek_slider.set(0)
        self.time_total.configure(text=self.format_time(self.song_length))
        
        pygame.mixer.music.load(song_path)
        pygame.mixer.music.play()
        
        self.is_paused = False
        self.core_play_btn.configure(text="⏸")
        self.big_play_btn.configure(text="⏸")
        self.track_name_lbl.configure(text=os.path.basename(song_path)[:22] + "...")
        
        self.update_slider_timer()

    def update_slider_timer(self):
        if pygame.mixer.music.get_busy() and not self.is_paused and not self.is_dragging:
            pos_di_pemutar = pygame.mixer.music.get_pos() / 1000
            if pos_di_pemutar >= 0:
                self.current_time = pos_di_pemutar
                self.seek_slider.set(self.current_time)
                self.time_passed.configure(text=self.format_time(self.current_time))
            
            if self.current_time >= self.song_length - 1:
                self.next_song()
                return
                
        if pygame.mixer.music.get_busy() and not self.is_paused:
            self.after(500, self.update_slider_timer)

    def on_slider_press(self, event):
        self.is_dragging = True

    def on_slider_release(self, event):
        if self.current_song_index is not None:
            self.current_time = int(self.seek_slider.get())
            pygame.mixer.music.play(start=self.current_time)
            if self.is_paused: pygame.mixer.music.pause()
            self.is_dragging = False
            self.update_slider_timer()

    def toggle_play(self):
        if self.current_song_index is None and self.playlist:
            self.play_song(0)
        elif self.current_song_index is not None:
            if self.is_paused:
                pygame.mixer.music.unpause()
                self.is_paused = False
                self.core_play_btn.configure(text="⏸")
                self.big_play_btn.configure(text="⏸")
                self.update_slider_timer()
            else:
                pygame.mixer.music.pause()
                self.is_paused = True
                self.core_play_btn.configure(text="▶️")
                self.big_color_btn = self.big_play_btn.configure(text="▶️")

    def next_song(self):
        if self.current_song_index is not None:
            self.play_song((self.current_song_index + 1) % len(self.playlist))

    def prev_song(self):
        if self.current_song_index is not None:
            self.play_song((self.current_song_index - 1) % len(self.playlist))

if __name__ == "__main__":
    app = PremiumSpotifyClone()
    app.mainloop()
