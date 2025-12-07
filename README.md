{ config, pkgs, ... }:

{
  imports =
    [ 
      ./hardware-configuration.nix
    ];

  # --- ЗАГРУЗЧИК (GRUB для портативного SSD) ---
  boot.loader.systemd-boot.enable = false;
  boot.loader.grub = {
    enable = true;
    device = "nodev";
    efiSupport = true;
    efiInstallAsRemovable = true; # Позволяет грузиться на любом ПК
  };
  # ВАЖНО: Запрещаем трогать BIOS хост-компьютера, чтобы избежать ошибок установки
  boot.loader.efi.canTouchEfiVariables = false;

  # --- ДИСКИ (По меткам, надежно для USB) ---
  fileSystems."/" = {
    device = "/dev/disk/by-label/NIXOS";
    fsType = "ext4";
  };

  fileSystems."/boot" = {
    device = "/dev/disk/by-label/BOOT";
    fsType = "vfat";
  };

  # --- СЕТЬ ---
  networking.hostName = "nixos-portable";
  networking.networkmanager.enable = true;

  # --- ЛОКАЛЬ И ВРЕМЯ ---
  time.timeZone = "Europe/Moscow";
  i18n.defaultLocale = "en_US.UTF-8";

  # --- ОПТИМИЗАЦИЯ ---
  services.fstrim.enable = true; # Продлевает жизнь SSD

  # --- ПОЛЬЗОВАТЕЛЬ ---
  users.users.user = {
    isNormalUser = true;
    description = "User";
    extraGroups = [ "networkmanager" "wheel" "video" "input" ];
    initialPassword = "1"; # Временный пароль "1"
    packages = with pkgs; [
      firefox
      alacritty # Терминал
      git
      wget
      fuzzel    # Меню запуска
    ];
  };

  # --- NIRI (Оконный менеджер) ---
  programs.niri.enable = true; # programs (множественное число!)

  # --- ГРАФИКА ---
  hardware.graphics.enable = true;

  # --- ЗВУК (Pipewire) ---
  security.rtkit.enable = true;
  services.pipewire = { # services (множественное число!)
    enable = true;
    alsa.enable = true;
    alsa.support32Bit = true;
    pulse.enable = true;
  };

  # --- СИСТЕМНЫЕ ПАКЕТЫ ---
  environment.systemPackages = with pkgs; [
    vim
    wget
    git
    waybar
  ];

  system.stateVersion = "25.11";
}
