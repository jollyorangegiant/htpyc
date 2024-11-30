#Version 0.1.1
#11/29/24
#Cameron McEwen

#CONFIG VARIABLES can be set here or in htpyc.json if it exists
#htpyc.json will override settings here
config = {}
config["defaultPlayer"] = "" #Overriding setting: string containing the launch string for a given supported player (MPV/VLC)
config["defaultLaunchString"] = "" #Dependent setting: string containing the full launch command for an unsupported player eg "mplayer -fs"
#END CONFIG VARIABLES

import pygame # type: ignore
import subprocess
import sys
import os
import platform
import threading
import time
from psutil import process_iter # type: ignore
import re
import json

#if a config file exists load it's values into the config dictionary
if os.path.exists("htpyc.json"):
    with open("htpyc.json", "r") as file:
        config = json.load(file)


# Initialize Pygame
pygame.init()

# Set the display to fullscreen
screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
screen_width, screen_height = pygame.display.get_surface().get_size()
pygame.display.set_caption('MPV Media Player Launcher')

# Define colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GRAY = (200, 200, 200)
BLUE = (0, 122, 204)
RED = (204, 0, 0)

# Header bar properties
header_height = int(screen_height * 0.05)  # Reduced to 5% of the screen height
header_color = BLACK

# Button properties
button_width = screen_width * 0.2  # 20% of screen width
button_height = 100
button_margin = 30
button_border_radius = 30  # Rounded corners

# Define supported media file types
MEDIA_EXTENSIONS = ['.avi', '.mkv', '.m4v']

# Time between valid clicks for debouncing
debounce_time = 0.5  # 500ms
last_click_time = 0

# Directory navigation history
directory_history = [os.getcwd()]

#we'll assume for now that all file words are deliniated by space . or ,
def is_space_or_dot_separated(s):
    if re.fullmatch(r"(?:\w+ )+\w+", s):
        return " "
    #elif re.fullmatch(r"(?:\w+\.)+\w+", s):
    #    return "."
    else:
        return "."

# OS-Specific MPV Command
def get_media_command():
    #current_os = platform.system()

    playerString = "vlc"
    args = ["--fullscreen", "--play-and-exit"]

    #a custom command string can override the out of the box behavior unless a supported defaultPlayer is set
    if config["defaultLaunchString"] != "":
        split = config["defaultLaunchString"].split(" ")
        argCount = 0
        args = []
        playerString = split[argCount]
        argCount +=1
        while argCount < len(split):
            print(split[argCount])
            args.append
            argCount +=1

    #the defaultPlayer can override a command string but will only change here if it's a supported player
    if config["defaultPlayer"] != "" and (config["defaultPlayer"].lower() == "vlc" or config["defaultPlayer"].lower() == "mpv"):
        playerString = config["defaultPlayer"]
        args = ["--fullscreen"]
        if config["defaultPlayer"].lower() == "vlc":
            args.append("--play-and-exit")
        else:
            print("no support for --quit in mpv")
    
    returnValue = [playerString]
    for argument in args:
        returnValue.append(argument)
    return returnValue

def is_media_player_running():
    """Check if MPV is already running."""
    for proc in process_iter(attrs=['name']):
        if proc.info['name'] == 'mpv' or proc.info['name'] == 'vlc':
            return True
    return False

def launch_media_player(media_file):
    """Launch a media player with the media file."""
    if is_media_player_running():
        print("Media Player is already running, skipping new launch.")
        return
    
    media_command = get_media_command()
    try:
        subprocess.run(media_command + [media_file], check=True)
    except subprocess.CalledProcessError as e:
        print(f"Error launching " + media_command + " {e}")
    except FileNotFoundError:
        print("MPV is not installed or not in PATH.")

# Threaded video launch function
def launch_video_in_thread(media_file):
    video_thread = threading.Thread(target=launch_media_player, args=(media_file,))
    video_thread.start()

def draw_text(surface, text, color, rect, font, aa=True, bkg=None):
    """Draw text with wrapping, scaling, and hyphenation."""
    rect = pygame.Rect(rect)
    y = rect.top
    line_spacing = -2

    def wrap_or_hyphenate(text, font, rect_width):
        """Wrap text into lines, hyphenating if necessary."""
        words = text.split(' ')
        space = font.size(' ')[0]
        lines = []
        if len(words) == 1:  # No spaces, need to hyphenate
            long_word = words[0]
            while len(long_word) > 0:
                chunk = long_word[:max(1, rect_width // (font.size("A")[0])) - 1] + "-"
                if len(chunk) >= len(long_word):  # No need for a hyphen at the end
                    chunk = long_word
                lines.append(chunk)
                long_word = long_word[len(chunk.rstrip("-")):]
        else:
            while words:
                line = []
                while words and font.size(' '.join(line + [words[0]]))[0] <= rect_width:
                    line.append(words.pop(0))
                if not line:  # Force at least one word per line
                    line.append(words.pop(0))
                lines.append(' '.join(line))
        return lines

    # Start with the current font size
    original_font_size = font.get_height()
    current_font_size = original_font_size

    # Attempt to fit text by reducing font size
    while True:
        font = pygame.font.Font(None, current_font_size)
        lines = wrap_or_hyphenate(text, font, rect.width)

        # Calculate total height of the text block
        total_text_height = len(lines) * (font.size("Tg")[1] + line_spacing)

        if total_text_height <= rect.height and all(font.size(line)[0] <= rect.width for line in lines):
            break  # Text fits
        current_font_size -= 1
        if current_font_size < 10:  # Minimum readable size
            break

    # Center the text vertically within the button
    y = rect.top + (rect.height - total_text_height) // 2

    # Render each line
    for line in lines:
        if y + font.size(line)[1] >= rect.bottom:
            break
        if bkg:
            image = font.render(line, aa, color, bkg)
            image.set_colorkey(bkg)
        else:
            image = font.render(line, aa, color)
        surface.blit(image, (rect.left + (rect.width - image.get_width()) // 2, y))  # Center horizontally
        y += font.size(line)[1] + line_spacing


def draw_button(text, rect, color):
    """Draws a button with rounded corners and scaled text inside."""
    pygame.draw.rect(screen, color, rect, border_radius=button_border_radius)
    font = pygame.font.Font(None, 48)  # Default starting font size
    draw_text(screen, text, WHITE, rect, font)


def draw_header(current_directory_cleaned):
    """Draws the header bar with back, home, and exit buttons, and displays the current directory."""
    # Draw the header background
    pygame.draw.rect(screen, header_color, (0, 0, screen_width, header_height))

    # Draw the exit button ("X")
    exit_button_width = int(header_height * 0.8)
    exit_button_rect = pygame.Rect(screen_width - exit_button_width - 10, (header_height - exit_button_width) // 2, exit_button_width, exit_button_width)
    pygame.draw.rect(screen, RED, exit_button_rect)

    font = pygame.font.Font(None, int(exit_button_width * 0.8))
    text_surface = font.render("X", True, WHITE)
    text_rect = text_surface.get_rect(center=exit_button_rect.center)
    screen.blit(text_surface, text_rect)

    # Draw the back button ("<")
    back_button_rect = pygame.Rect(10, (header_height - exit_button_width) // 2, exit_button_width, exit_button_width)
    pygame.draw.rect(screen, BLUE, back_button_rect)
    text_surface = font.render("<", True, WHITE)
    text_rect = text_surface.get_rect(center=back_button_rect.center)
    screen.blit(text_surface, text_rect)

    # Draw the home button ("H")
    home_button_rect = pygame.Rect(70, (header_height - exit_button_width) // 2, exit_button_width, exit_button_width)
    pygame.draw.rect(screen, BLUE, home_button_rect)
    text_surface = font.render("H", True, WHITE)
    text_rect = text_surface.get_rect(center=home_button_rect.center)
    screen.blit(text_surface, text_rect)

    # Display the current directory or "Home"
    header_font = pygame.font.Font(None, int(header_height * 0.8))
    text = current_directory_cleaned if current_directory_cleaned else "Home"
    text_surface = header_font.render(text, True, WHITE)
    text_rect = text_surface.get_rect(center=(screen_width // 2, header_height // 2))
    screen.blit(text_surface, text_rect)

    return exit_button_rect, back_button_rect, home_button_rect

def find_media_and_dirs():
    """Scans the current directory for media files and subdirectories."""
    media_files = []
    for item in os.listdir(os.getcwd()):
        item_path = os.path.join(os.getcwd(), item)
        # Add directories or files with the supported extensions
        if os.path.isdir(item_path) or any(item.endswith(ext) for ext in MEDIA_EXTENSIONS):
            media_files.append(item)
    return media_files

def find_media_only():
    """Scans the current directory for media files and subdirectories."""
    media_files = []
    for item in os.listdir(os.getcwd()):
        item_path = os.path.join(os.getcwd(), item)
        # Add directories or files with the supported extensions
        if any(item.endswith(ext) for ext in MEDIA_EXTENSIONS):
            media_files.append(item)
    return media_files

def clean_filename(filename, parent_dir):
    """Clean the filename by removing the immediate parent directory name (case-insensitive),
       normalizing season/episode notation, and removing quality/encoding identifiers."""
    quality_and_encoding_patterns = [
        r'\b(1080p|720p|480p|4k|hdr|hevc|x264|x265|avi|mp4|mkv|mov|flv|wmv|bluray|aac)\b'
    ]
    
    season_episode_patterns = [
        r'\bs(\d{1,2})e(\d{1,2})\b',  # S01E01, S1E1
        r'\bseason[\.\s]*(\d{1,2})[\.\s]*episode[\.\s]*(\d{1,2})\b',  # Season 1 Episode 1
        r'\bseason[\.\s]*(\d{1,2})[\.\s]*ep[\.\s]*(\d{1,2})\b',  # Season 1 Ep 1
        r'\bseason[\.\s]*(\d{1,2})[\.\s]*e(\d{1,2})\b',  # Season 1 E1
    ]
    
    # Remove file extension
    filename_no_ext = os.path.splitext(filename)[0].lower()

    # Extract the immediate parent directory (ignoring the uppermost directory)
    parent_dir_clean = parent_dir.lower()

    # Check if the input is a directory
    if os.path.isdir(filename):
        # For directories: clean by removing the parent directory name (case-insensitive)
        filename_clean = filename_no_ext.lower().replace(parent_dir_clean, "").strip().replace("/", "").replace(".", " ")
        for pattern in season_episode_patterns:
            filename_clean = re.sub(pattern, r'S\1E\2', filename_clean)  # Normalize to S01E01 format
        for pattern in quality_and_encoding_patterns:
            filename_clean = re.sub(pattern, "", filename_clean).strip()

        return filename_clean.title()

    #detect if we're running in a "Season N" directory and use it's parent directory name if we are
    seasonPattern = r'(?i)^(?:season|s)\s*0?(\d{1,2})$'
    if re.search(seasonPattern, parent_dir_clean):
        #upperDir = parent_dir_clean + "/.."
        #upperDir = os.path.normpath(upperDir)
        currentDir = os.getcwd()
        parentDir = os.path.join(currentDir, "..")
        #print(f"With .. appended: {parentDir}")
        resolvedParentDir = os.path.abspath(parentDir)
        print(resolvedParentDir)
        #print(parent_dir_clean)
        parent_dir_clean = os.path.basename(resolvedParentDir).lower()
        print(parent_dir)

    print("Cleaning " + parent_dir_clean + " from " + filename_no_ext)
    #need to determine if space or dot deliniated
    parentDeliniator = is_space_or_dot_separated(parent_dir_clean)
    print("|" + parentDeliniator + "|")
    filenameDeliniator = is_space_or_dot_separated(filename_no_ext)
    print(filenameDeliniator)
    split_parent_dir = parent_dir_clean.split(parentDeliniator)
    split_filename = filename_no_ext.split(filenameDeliniator)

    #for media files, first check if it's the only media file in the directory
    #If it's the only file in the directory there's likely to be a string match file/directory IE S02E01
    #if that's the case "S02E01" should be left in the "cleaned" title
    mediaFiles = find_media_only()
    if len(mediaFiles) > 1:
        #filename_clean = filename_no_ext.lower().replace(parent_dir_clean, "")
        filename_clean = ""
        for word in split_filename:
            temp = word
            for search in split_parent_dir:
                #print(temp + ":" + search)
                if not re.match(r"^s\d{2}$", search):
                    temp = temp.replace(search,"")
            filename_clean = filename_clean + " " + temp

    else:
        filename_clean = ""
        for word in split_filename:
            temp = word
            for search in split_parent_dir:
                #print(temp + ":" + search)
                if not re.match(r"^s\d{2}e\d{2}$", search):
                    temp = temp.replace(search,"")
            filename_clean = filename_clean + " " + temp

    for pattern in season_episode_patterns:
        filename_clean = re.sub(pattern, r'S\1E\2', filename_clean)  # Normalize to S01E01 format
    for pattern in quality_and_encoding_patterns:
        filename_clean = re.sub(pattern, "", filename_clean).strip()

    # Remove excess dots or whitespace introduced during cleaning
    filename_clean = re.sub(r'\s+', ' ', filename_clean).strip()
    filename_clean = re.sub(r'\.+', '.', filename_clean).strip('.')

    return filename_clean

def truncate_filename(filename, max_length=30):
    """Truncate the filename if it's too long."""
    if len(filename) > max_length:
        return filename[:max_length-3] + "..."
    return filename

def debounce():
    """Handle click debounce to prevent multiple triggers."""
    global last_click_time
    current_time = time.time()
    if current_time - last_click_time > debounce_time:
        last_click_time = current_time
        return True
    return False

def navigate_directory(path):
    """Navigate to a directory and update the directory history."""
    os.chdir(path)
    directory_history.append(path)

def go_back():
    """Go back to the previous directory."""
    if len(directory_history) > 1:
        directory_history.pop()
        os.chdir(directory_history[-1])

# Main loop
def main():
    running = True
    media_files = find_media_and_dirs()
    buttons = []

    # Get the name of the current (parent) directory
    parent_directory_name = os.path.basename(os.getcwd())

    # Calculate the number of rows and columns for grid layout
    num_columns = 4  # You can adjust this number based on screen size
    num_rows = (len(media_files) + num_columns - 1) // num_columns  # Calculate rows based on number of files
    
    # Generate buttons for each media file or directory found
    for index, item in enumerate(media_files):
        item_path = os.path.join(os.getcwd(), item)
        cleaned_name = clean_filename(item, parent_directory_name)  # Clean the filename by removing parent directory match and extension
        truncated_name = truncate_filename(cleaned_name)
        
        # Calculate grid position
        row = index // num_columns
        col = index % num_columns
        button_x = button_margin + (button_width + button_margin) * col
        button_y = header_height + button_margin + (button_height + button_margin) * row  # Adjust for header height
        button_rect = pygame.Rect(button_x, button_y, button_width, button_height)
        buttons.append((button_rect, item_path, truncated_name))

    while running:
        screen.fill(GRAY)
        #print(directory_history[0])
        current_directory_cleaned = ""
        if os.getcwd() == directory_history[0]:
            current_directory_cleaned = "Home"
        else: 
            current_directory_cleaned = clean_filename(os.getcwd(), os.path.dirname(os.getcwd()))
        #print(current_directory_cleaned)
        # Draw the header
        exit_button_rect, back_button_rect, home_button_rect = draw_header(current_directory_cleaned)

        # Draw the buttons
        for button_rect, item_path, truncated_name in buttons:
            draw_button(truncated_name, button_rect, BLUE)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:  # Left mouse click only
                # Ensure debounce check before launching
                if debounce():
                    # Check if exit, back, or home buttons were clicked
                    if exit_button_rect.collidepoint(event.pos):
                        running = False
                    elif back_button_rect.collidepoint(event.pos):
                        go_back()
                        media_files = find_media_and_dirs()  # Rebuild the file list after directory change
                        buttons = []
                        for index, item in enumerate(media_files):
                            item_path = os.path.join(os.getcwd(), item)
                            cleaned_name = clean_filename(item, os.path.basename(os.getcwd()))  # Clean the filename
                            truncated_name = truncate_filename(cleaned_name)
                            row = index // num_columns
                            col = index % num_columns
                            button_x = button_margin + (button_width + button_margin) * col
                            button_y = header_height + button_margin + (button_height + button_margin) * row
                            button_rect = pygame.Rect(button_x, button_y, button_width, button_height)
                            buttons.append((button_rect, item_path, truncated_name))

                    elif home_button_rect.collidepoint(event.pos):
                        os.chdir(directory_history[0])  # Go to root
                        directory_history.clear()
                        directory_history.append(os.getcwd())
                        media_files = find_media_and_dirs()  # Rebuild the file list
                        buttons = []
                        for index, item in enumerate(media_files):
                            item_path = os.path.join(os.getcwd(), item)
                            cleaned_name = clean_filename(item, os.path.basename(os.getcwd()))  # Clean the filename
                            truncated_name = truncate_filename(cleaned_name)
                            row = index // num_columns
                            col = index % num_columns
                            button_x = button_margin + (button_width + button_margin) * col
                            button_y = header_height + button_margin + (button_height + button_margin) * row
                            button_rect = pygame.Rect(button_x, button_y, button_width, button_height)
                            buttons.append((button_rect, item_path, truncated_name))

                    # Check if a media button was clicked
                    for button_rect, item_path, truncated_name in buttons:
                        if button_rect.collidepoint(event.pos):
                            if os.path.isdir(item_path):
                                navigate_directory(item_path)
                                media_files = find_media_and_dirs()  # Rebuild the file list after directory change
                                buttons = []
                                for index, item in enumerate(media_files):
                                    item_path = os.path.join(os.getcwd(), item)
                                    cleaned_name = clean_filename(item, os.path.basename(os.getcwd()))  # Clean the filename
                                    truncated_name = truncate_filename(cleaned_name)
                                    row = index // num_columns
                                    col = index % num_columns
                                    button_x = button_margin + (button_width + button_margin) * col
                                    button_y = header_height + button_margin + (button_height + button_margin) * row
                                    button_rect = pygame.Rect(button_x, button_y, button_width, button_height)
                                    buttons.append((button_rect, item_path, truncated_name))
                            else:
                                launch_video_in_thread(item_path)

        pygame.display.flip()

    pygame.quit()

if __name__ == "__main__":
    main()
