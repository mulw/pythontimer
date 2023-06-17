import time
import tkinter as tk
from tkinter import messagebox, colorchooser, font

class TimerWindow:
    def __init__(self, root):
        self.root = root
        self.root.title("Timer")
        self.root.geometry("400x250")
        self.root.resizable(False, False)

        self.duration = 0
        self.start_time = 0
        self.end_time = 0
        self.timer_active = False

        self.timer_label = tk.Label(self.root, text="", font=("Arial", 36))
        self.timer_label.pack(pady=20)

        duration_frame = tk.Frame(self.root)
        duration_frame.pack(pady=10)

        self.duration_label = tk.Label(duration_frame, text="Duration (minutes):")
        self.duration_label.pack(side="left")

        self.duration_entry = tk.Entry(duration_frame)
        self.duration_entry.pack(side="left")

        button_frame = tk.Frame(self.root)
        button_frame.pack(pady=20)

        self.start_button = tk.Button(button_frame, text="Start", command=self.start_timer)
        self.start_button.pack(side="left", padx=10)

        self.stop_button = tk.Button(button_frame, text="Stop", command=self.stop_timer)
        self.stop_button.pack(side="left", padx=10)

        self.restart_button = tk.Button(button_frame, text="Restart", command=self.restart_timer)
        self.restart_button.pack(side="left", padx=10)

        settings_button = tk.Button(self.root, text="Settings", command=self.open_settings)
        settings_button.pack()

        self.timer_label.bind("<Enter>", self.show_buttons_on_hover)
        self.timer_label.bind("<Leave>", self.hide_buttons_on_leave)

        self.load_settings()

    def start_timer(self):
        if not self.timer_active:
            self.duration = int(self.duration_entry.get()) * 60
            if self.duration <= 0:
                messagebox.showerror("Invalid Duration", "Please enter a valid duration in minutes.")
                return

            self.start_time = time.time()
            self.end_time = self.start_time + self.duration
            self.timer_active = True
            self.update_timer()
            self.hide_elements()

    def stop_timer(self):
        self.timer_active = False
        self.timer_label.config(text="")
        self.show_elements()

    def restart_timer(self):
        self.stop_timer()
        self.start_timer()

    def update_timer(self):
        if self.timer_active:
            current_time = time.time()
            remaining_time = self.end_time - current_time

            if remaining_time <= 0:
                self.timer_active = False
                self.timer_label.config(text="")
                messagebox.showinfo("Time's up!", "The timer has ended.")
                return

            minutes, seconds = divmod(int(remaining_time), 60)
            hours, minutes = divmod(minutes, 60)

            timer_string = "{:02d}:{:02d}:{:02d}".format(hours, minutes, seconds)
            self.timer_label.config(text=timer_string)
            self.adjust_font_size()

        self.root.after(1000, self.update_timer)

    def open_settings(self):
        settings_window = tk.Toplevel(self.root)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
        settings_window.resizable(False, False)

        bg_color_frame = tk.Frame(settings_window)
        bg_color_frame.pack(pady=10)

        bg_color_label = tk.Label(bg_color_frame, text="Background Color:")
        bg_color_label.pack(side="left")

        bg_color_button = tk.Button(bg_color_frame, text="Select Color", command=self.select_bg_color)
        bg_color_button.pack(side="left", padx=10)

        font_color_frame = tk.Frame(settings_window)
        font_color_frame.pack(pady=10)

        font_color_label = tk.Label(font_color_frame, text="Font Color:")
        font_color_label.pack(side="left")

        font_color_button = tk.Button(font_color_frame, text="Select Color", command=self.select_font_color)
        font_color_button.pack(side="left", padx=10)

        font_type_frame = tk.Frame(settings_window)
        font_type_frame.pack(pady=10)

        font_type_label = tk.Label(font_type_frame, text="Font Type:")
        font_type_label.pack(side="left")

        self.font_type_variable = tk.StringVar(settings_window)
        self.font_type_variable.set("Arial")
        font_type_option = tk.OptionMenu(font_type_frame, self.font_type_variable, *font.families(), command=self.change_font_type)
        font_type_option.pack(side="left", padx=10)

        save_button = tk.Button(settings_window, text="Save Settings", command=self.save_settings)
        save_button.pack(pady=20)

        self.hide_elements()

    def select_bg_color(self):
        color = colorchooser.askcolor(title="Select Background Color")[1]
        if color:
            self.timer_label.configure(bg=color)

    def select_font_color(self):
        color = colorchooser.askcolor(title="Select Font Color")[1]
        if color:
            self.timer_label.configure(fg=color)

    def change_font_type(self, selected_font):
        current_font = font.Font(family=selected_font)
        self.timer_label.configure(font=current_font)

    def show_buttons_on_hover(self, event):
        self.show_elements()

    def hide_buttons_on_leave(self, event):
        if not self.timer_active:
            self.hide_elements()

    def hide_elements(self):
        self.duration_label.pack_forget()
        self.duration_entry.pack_forget()
        self.start_button.pack_forget()
        self.stop_button.pack_forget()
        self.restart_button.pack_forget()

    def show_elements(self):
        self.duration_label.pack(side="left")
        self.duration_entry.pack(side="left")
        self.start_button.pack(side="left", padx=10)
        self.stop_button.pack(side="left", padx=10)
        self.restart_button.pack(side="left", padx=10)

    def adjust_font_size(self):
        timer_text = self.timer_label.cget("text")
        current_font = self.timer_label.cget("font")
        font_size = 36

        while True:
            font_config = font.Font(font=current_font)
            text_width = font_config.measure(timer_text)

            if text_width <= self.root.winfo_width() - 20:
                self.timer_label.config(font=("Arial", font_size))
                break

            font_size -= 1

    def save_settings(self):
        bg_color = self.timer_label.cget("bg")
        font_color = self.timer_label.cget("fg")
        font_type = self.font_type_variable.get()

        with open("timer_settings.txt", "w") as f:
            f.write(bg_color + "\n")
            f.write(font_color + "\n")
            f.write(font_type + "\n")

    def load_settings(self):
        try:
            with open("timer_settings.txt", "r") as f:
                bg_color = f.readline().strip()
                font_color = f.readline().strip()
                font_type = f.readline().strip()

                self.timer_label.configure(bg=bg_color, fg=font_color)
                current_font = font.Font(family=font_type)
                self.timer_label.configure(font=current_font)
                self.font_type_variable.set(font_type)
        except FileNotFoundError:
            pass

root = tk.Tk()
timer_window = TimerWindow(root)
root.mainloop()
