import os
import time
import requests
from rich.console import Console
from rich.panel import Panel
from rich.prompt import Prompt, Confirm
from rich.progress import Progress, SpinnerColumn, BarColumn, TextColumn, TimeRemainingColumn
import inquirer
from pathlib import Path

console = Console()

ASCII_ART = r"""
[bold cyan]
   _    ____ ___   _  __          __   __    _ _     _       _
  /_\  |  _ \_ _| | |/ /___ _   __\ \ / /_ _| (_) __| | __ _| |_ ___  _ __
//_\\ | |_) | |  | ' // _ \ | | __\ V / _` | | |/ _` |/ _` | __/ _ \| '__|
/  _  \|  __/| |  | . \  __/ |_| |_ | | (_| | | | (_| | (_| | || (_) | |
\_/ \_/|_|  |___| |_|\_\___|\__, (_)|_|\__,_|_|_|\__,_|\__,_|\__\___/|_|
                            |___/
[/bold cyan]
"""

def get_api_key_from_env_or_file(env_var, file_path):
    api_key = os.getenv(env_var)
    if api_key:
        console.print(f"[cyan]Using API key from environment variable:[/cyan] {env_var}")
        return api_key
    elif os.path.exists(file_path):
        with open(file_path, 'r') as file:
            api_key = file.read().strip()
            console.print(f"[cyan]Using API key from file:[/cyan] {file_path}")
            return api_key
    else:
        console.print(f"[red]No API key found in environment variable {env_var} or file {file_path}.[/red]")
        return None

def parse_api_key(line, api_choice):
    if api_choice == 'Heroku API Key' and line.startswith('Heroku API KEY  ->'):
        return line.split('->')[1].strip()
    elif api_choice == 'Google Maps API Key' and line.startswith('google_api      ->'):
        return line.split('->')[1].strip()
    return None

def check_heroku_key(api_key):
    url = "https://api.heroku.com/apps"
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Accept": "application/vnd.heroku+json; version=3"
    }
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        return True, url
    else:
        console.print(f"[red]Heroku API Error: {response.status_code} - {response.text}[/red]")
        return False, url

def check_google_maps_api_key(api_key):
    url = "https://maps.googleapis.com/maps/api/staticmap"
    params = {
        "center": "40.714728,-73.998672",
        "zoom": "12",
        "size": "2500x2000",
        "maptype": "roadmap",
        "key": api_key
    }
    response = requests.get(url, params=params)
    if response.status_code == 200:
        return True, response.url
    else:
        console.print(f"[red]Google Maps API Error: {response.status_code} - {response.text}[/red]")
        return False, response.url

def validate_keys(api_key, check_function):
    is_valid, url = check_function(api_key)
    if is_valid:
        console.print(Panel(f"[green]API Key is valid.[/green]\n[bold]URL:[/bold] {url}", title="Success", style="green"))
        return True, url
    else:
        console.print(Panel("[red]API Key is invalid.[/red]", title="Error", style="red"))
        return False, url

def save_valid_keys(valid_keys, file_name):
    with open(file_name, 'w') as file:
        for key_type, key, url in valid_keys:
            file.write(f"{key_type}: {key}\nURL: {url}\n\n")
    console.print(f"[green]Valid keys saved to {file_name}[/green]")

def suggest_path(text, state):
    current_path = Path(text)
    if current_path.is_dir():
        suggestions = [str(p) for p in current_path.glob('*') if p.name.startswith(text.split(os.sep)[-1])]
    else:
        suggestions = [str(p) for p in current_path.parent.glob(f'{current_path.name}*')]
    return suggestions[state] if state < len(suggestions) else None

def main():
    console.print(ASCII_ART)
    console.print(Panel("[bold magenta]API Key Validator[/bold magenta]", expand=False))
    questions = [
        inquirer.List('api_choice',
                      message="Select the API to validate",
                      choices=['Heroku API Key', 'Google Maps API Key', 'All'],
                      ),
        inquirer.List('input_type',
                      message="Do you want to enter a single API key, use environment variables, or a file containing API keys?",
                      choices=['single', 'env', 'file'],
                      default='single'
                      )
    ]
    answers = inquirer.prompt(questions)

    valid_keys = []

    if answers['input_type'] == 'single':
        if answers['api_choice'] == 'All':
            heroku_key = Prompt.ask("[bold]Enter the Heroku API Key:[/bold]")
            google_maps_key = Prompt.ask("[bold]Enter the Google Maps API Key:[/bold]")
            with Progress(SpinnerColumn(), TextColumn("[progress.description]{task.description}")) as progress:
                task = progress.add_task("Validating All API Keys...", start=False)
                progress.start_task(task)
                validate_keys(heroku_key, check_heroku_key)
                validate_keys(google_maps_key, check_google_maps_api_key)
                progress.stop_task(task)
        else:
            api_key = Prompt.ask("[bold]Enter the API key:[/bold]")
            with Progress(SpinnerColumn(), TextColumn("[progress.description]{task.description}")) as progress:
                task = progress.add_task("Validating API Key...", start=False)
                progress.start_task(task)
                if answers['api_choice'] == 'Heroku API Key':
                    validate_keys(api_key, check_heroku_key)
                elif answers['api_choice'] == 'Google Maps API Key':
                    validate_keys(api_key, check_google_maps_api_key)
                progress.stop_task(task)
    elif answers['input_type'] == 'env':
        if answers['api_choice'] == 'Heroku API Key':
            api_key = get_api_key_from_env_or_file('HEROKU_KEY', 'heroku_key.txt')
            if api_key:
                validate_keys(api_key, check_heroku_key)
        elif answers['api_choice'] == 'Google Maps API Key':
            api_key = get_api_key_from_env_or_file('GOOGLE_MAPS_API_KEY', 'google_maps_key.txt')
            if api_key:
                validate_keys(api_key, check_google_maps_api_key)
        elif answers['api_choice'] == 'All':
            heroku_key = get_api_key_from_env_or_file('HEROKU_KEY', 'heroku_key.txt')
            google_maps_key = get_api_key_from_env_or_file('GOOGLE_MAPS_API_KEY', 'google_maps_key.txt')
            if heroku_key:
                validate_keys(heroku_key, check_heroku_key)
            if google_maps_key:
                validate_keys(google_maps_key, check_google_maps_api_key)

    elif answers['input_type'] == 'file':
        file_path = Prompt.ask("[bold]Enter the path to the file containing API keys:[/bold]")
        try:
            with open(file_path, 'r') as file:
                lines = file.readlines()
                if answers['api_choice'] == 'All':
                    heroku_keys = [parse_api_key(line, 'Heroku API Key') for line in lines if parse_api_key(line, 'Heroku API Key')]
                    google_maps_keys = [parse_api_key(line, 'Google Maps API Key') for line in lines if parse_api_key(line, 'Google Maps API Key')]
                    total_keys = len(heroku_keys) + len(google_maps_keys)
                    console.print(f"[cyan]Total keys to scan: {total_keys}[/cyan]")
                    with Progress(SpinnerColumn(), BarColumn(), TextColumn("[progress.description]{task.description}"), TimeRemainingColumn()) as progress:
                        task = progress.add_task("Scanning API Keys...", total=total_keys)
                        for i, api_key in enumerate(heroku_keys + google_maps_keys, start=1):
                            if api_key:
                                if api_key in heroku_keys:
                                    console.print(f"[blue]Checking Heroku API Key:[/blue] {api_key}")
                                    is_valid, url = validate_keys(api_key, check_heroku_key)
                                    if is_valid:
                                        valid_keys.append(('Heroku API Key', api_key, url))
                                else:
                                    console.print(f"[blue]Checking Google Maps API Key:[/blue] {api_key}")
                                    is_valid, url = validate_keys(api_key, check_google_maps_api_key)
                                    if is_valid:
                                        valid_keys.append(('Google Maps API Key', api_key, url))
                            progress.update(task, advance=1, description=f"Scanning API Keys... ({i}/{total_keys})")
                            time.sleep(0.1)  # Simulate processing time
                else:
                    relevant_keys = [parse_api_key(line, answers['api_choice']) for line in lines if parse_api_key(line, answers['api_choice'])]
                    total_keys = len(relevant_keys)
                    console.print(f"[cyan]Total {answers['api_choice']} keys to scan: {total_keys}[/cyan]")
                    with Progress(SpinnerColumn(), BarColumn(), TextColumn("[progress.description]{task.description}"), TimeRemainingColumn()) as progress:
                        task = progress.add_task("Scanning API Keys...", total=total_keys)
                        for i, api_key in enumerate(relevant_keys, start=1):
                            if api_key:
                                if answers['api_choice'] == 'Heroku API Key':
                                    console.print(f"[blue]Checking Heroku API Key:[/blue] {api_key}")
                                    is_valid, url = validate_keys(api_key, check_heroku_key)
                                    if is_valid:
                                        valid_keys.append(('Heroku API Key', api_key, url))
                                elif answers['api_choice'] == 'Google Maps API Key':
                                    console.print(f"[blue]Checking Google Maps API Key:[/blue] {api_key}")
                                    is_valid, url = validate_keys(api_key, check_google_maps_api_key)
                                    if is_valid:
                                        valid_keys.append(('Google Maps API Key', api_key, url))
                            progress.update(task, advance=1, description=f"Scanning API Keys... ({i}/{total_keys})")
                            time.sleep(0.1)  # Simulate processing time
        except FileNotFoundError:
            console.print(Panel("[red]File not found. Please check the path and try again.[/red]", title="Error", style="red"))

    console.print(f"[green]Total valid API keys found: {len(valid_keys)}[/green]")

    if valid_keys:
        save_option = Confirm.ask("Do you want to save the valid API keys to a file?")
        if save_option:
            file_name = Prompt.ask("Enter the file name to save the valid keys", default="valid_api_keys.txt")
            save_valid_keys(valid_keys, file_name)

if __name__ == "__main__":
    main()
