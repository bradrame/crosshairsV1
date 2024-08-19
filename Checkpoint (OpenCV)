import ctypes
import threading
import pyautogui
import tkinter as tk
import cv2
import numpy as np
from PIL import Image, ImageTk

# Required function
def capture_screen():
    screen_width, screen_height = pyautogui.size()
    screenshot = pyautogui.screenshot()
    screenshot = cv2.cvtColor(np.array(screenshot), cv2.COLOR_RGB2BGR)
    return screenshot, screen_width, screen_height

# Variables
screenshot, screen_width, screen_height = capture_screen()
center_x, center_y = screen_width // 2, screen_height // 2
zoomed_in = False
magnify_factor = 2
magnifier_radius = 200
initial_x = screen_width // 2 - magnifier_radius
initial_y = 25

# App settings
app = tk.Tk()
app.geometry(f'{screen_width}x{screen_height}')
app.attributes('-topmost', True)
app.overrideredirect(1)
app.wm_attributes('-transparentcolor', app['bg'])

crosshair_window = tk.Toplevel(app)
crosshair_window.geometry(f'{screen_width}x{screen_height}')
crosshair_window.attributes('-topmost', True)
crosshair_window.overrideredirect(1)
crosshair_window.wm_attributes('-transparentcolor', crosshair_window['bg'])

scope_window = tk.Toplevel(app)
scope_window.geometry(f'{magnifier_radius * magnify_factor}x{magnifier_radius * magnify_factor}+{initial_x}+{initial_y}')
scope_window.attributes('-topmost', True)
scope_window.overrideredirect(1)
scope_window.wm_attributes('-transparentcolor', app['bg'])

app.update_idletasks()
hwnd_app = ctypes.windll.user32.GetParent(app.winfo_id())
hwnd_crosshair = ctypes.windll.user32.GetParent(crosshair_window.winfo_id())
extended_style = ctypes.windll.user32.GetWindowLongW(hwnd_crosshair, -20)
ctypes.windll.user32.SetWindowLongW(hwnd_crosshair, -20, extended_style | 0x80000 | 0x20)
ctypes.windll.user32.SetWindowPos(hwnd_crosshair, hwnd_app, 0, 0, 0, 0, 0x0003)

def magnify():
    def get_magnified_region(screenshot, center_x, center_y, magnify_factor, region_size):
        left = max(center_x - region_size // 2, 0)
        top = max(center_y - region_size // 2, 0)
        right = min(center_x + region_size // 2, screenshot.shape[1])
        bottom = min(center_y + region_size // 2, screenshot.shape[0])

        region = screenshot[top:bottom, left:right]
        magnified_region = cv2.resize(region, (region_size * magnify_factor, region_size * magnify_factor), interpolation=cv2.INTER_LANCZOS4)
        return magnified_region

    def create_circular_image(image):
        height, width, _ = image.shape
        mask = np.zeros((height, width), dtype=np.uint8)
        cv2.circle(mask, (width // 2, height // 2), min(width, height) // 2, 255, -1)
        bgra_image = cv2.cvtColor(image, cv2.COLOR_BGR2BGRA)
        bgra_image[:, :, 3] = mask
        return bgra_image

    def update_magnified_image():
        global screenshot
        screenshot, _, _ = capture_screen()
        center_x, center_y = pyautogui.position()
        magnified_region = get_magnified_region(screenshot, center_x, center_y, magnify_factor, magnifier_radius)
        circular_magnified_region = create_circular_image(magnified_region)
        circular_magnified_region = Image.fromarray(cv2.cvtColor(circular_magnified_region, cv2.COLOR_BGRA2RGBA))
        magnified_photo = ImageTk.PhotoImage(circular_magnified_region)
        label.config(image=magnified_photo)
        label.image = magnified_photo
        if zoomed_in == True:
            scope_window.after(1, update_magnified_image)
        else:
            label.destroy()

    magnified_region = get_magnified_region(screenshot,
                                            screen_width // 2,
                                            screen_height // 2,
                                            magnify_factor,
                                            magnifier_radius)
    circular_magnified_region = create_circular_image(magnified_region)
    circular_magnified_region = Image.fromarray(cv2.cvtColor(circular_magnified_region, cv2.COLOR_BGRA2RGBA))
    magnified_photo = ImageTk.PhotoImage(circular_magnified_region)

    label = tk.Label(scope_window, image=magnified_photo, bg=app['bg'])
    label.pack()
    if zoomed_in == True:
        scope_window.after(1, update_magnified_image)

def load_crosshair():
    global crosshair_canvas, crosshair_item
    crosshair_canvas = tk.Canvas(crosshair_window, width=screen_width, height=screen_height, bg=crosshair_window['bg'], highlightthickness=0)
    crosshair_canvas.place(x=0, y=0)
    crosshair_item = crosshair_canvas.create_image(center_x, center_y, image=crosshair_image)

def button_press(button_input):
    global center_x, center_y, zoomed_in, crosshair_image
    if button_input == 'left':
        center_x -= 1
    elif button_input == 'right':
        center_x += 1
    elif button_input == 'up':
        center_y -= 1
    elif button_input == 'down':
        center_y += 1
    elif button_input == 'in':
        zoomed_in = True
        zoom_in_button.config(state=tk.DISABLED)
        app.bind('<Up>', lambda event: None)
        app.bind('<Down>', lambda event: threaded_button_press('out'))
        zoom_out_button.config(state=tk.NORMAL)
        magnify()
    elif button_input == 'out':
        zoomed_in = False
        zoom_out_button.config(state=tk.DISABLED)
        app.bind('<Down>', lambda event: None)
        app.bind('<Up>', lambda event: threaded_button_press('in'))
        zoom_in_button.config(state=tk.NORMAL)
    elif button_input == 'show':
        crosshair_button.config(text='Hide crosshair', command=lambda: threaded_button_press('hide'))
        left_button.config(state=tk.NORMAL)
        up_button.config(state=tk.NORMAL)
        right_button.config(state=tk.NORMAL)
        down_button.config(state=tk.NORMAL)
        crosshair_image = tk.PhotoImage(file='crosshair1.png')
        load_crosshair()
    elif button_input == 'hide':
        crosshair_button.config(text='Show crosshair', command=lambda: threaded_button_press('show'))
        left_button.config(state=tk.DISABLED)
        up_button.config(state=tk.DISABLED)
        right_button.config(state=tk.DISABLED)
        down_button.config(state=tk.DISABLED)
        crosshair_image = tk.PhotoImage(file='')
        load_crosshair()
    elif button_input == 'quit':
        crosshair_window.withdraw()
        app.quit()
    if 'crosshair_canvas' in globals():
        crosshair_canvas.coords(crosshair_item, center_x, center_y)

def threaded_button_press(button_input):
    print(f'You pressed {button_input}')
    thread = threading.Thread(target=button_press, args=(button_input,))
    thread.start()

frame = tk.Frame(app, bg=app['bg'])
frame.place(relx=0.5, rely=0, anchor='n')

left_button = tk.Button(frame, text='←', bg='white', command=lambda: threaded_button_press('left'), state=tk.DISABLED)
left_button.grid(row=0, column=0)
up_button = tk.Button(frame, text='↑', bg='white', command=lambda: threaded_button_press('up'), state=tk.DISABLED)
up_button.grid(row=0, column=1)
app.bind('<Up>', lambda event: threaded_button_press('in'))
right_button = tk.Button(frame, text='→', bg='white', command=lambda: threaded_button_press('right'), state=tk.DISABLED)
right_button.grid(row=0, column=2)
down_button = tk.Button(frame, text='↓', bg='white', command=lambda: threaded_button_press('down'), state=tk.DISABLED)
down_button.grid(row=0, column=3)
crosshair_button = tk.Button(frame, text='Show crosshair', bg='white', command=lambda: threaded_button_press('show'))
crosshair_button.grid(row=0, column=4)
quit_button = tk.Button(frame, text='Quit', bg='white', command=lambda: threaded_button_press('quit'))
quit_button.grid(row=0, column=5)
zoom_in_button = tk.Button(frame, text='+', bg='white', command=lambda: threaded_button_press('in'))
zoom_in_button.grid(row=0, column=6)
zoom_out_button = tk.Button(frame, text='-', bg='white', command=lambda: threaded_button_press('out'), state=tk.DISABLED)
zoom_out_button.grid(row=0, column=7)

app.focus_force()
app.mainloop()