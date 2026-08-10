# Creating a virtual network adapter with the desired MAC address

## 1. Introduction

### What is a MAC Address?
MAC address (Media Access Control) is a unique physical identifier assigned to network hardware (NICs, Wi-Fi modules, routers) by the manufacturer. It acts as a device "passport" at the data link layer.

## 2. Usage Scenarios

### Administration (ISP and Router Replacement)
Your home or office ISP binds your internet contract to the MAC address of the original connected hardware (old PC or router), and you bought a new one. **Goal:** Restore internet access instantly by cloning the old MAC address onto the new hardware, avoiding delays and calls to technical support.

### Testing, Security and Privacy
You need to bypass corporate/public MAC-filtering, test network equipment, or prevent tracking in public spaces. **Goal:** Connect unauthorized personal devices to whitelisted networks, simulate multi-client loads during QA testing, or mask your real identity from trackers in public Wi-Fi.

## 3. Windows Instructions

### Method 1: Physical Adapter Masking
Directly modifying the settings of your actual network card.

1. **Device Manager:** Right-click Start -> Device Manager.
2. **Find Adapter:** Expand "Network adapters" -> Right-click your card -> Properties.
3. **Advanced Tab:** Select "Network Address" (or "Locally Administered Address").
4. **Input:** Type the MAC without separators (e.g., `102ABB3C4D5F`).
5. **Registry Method:** If missing, go to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4D36E972-E325-11CE-BFC1-08002BE10318}` and add a String Value `NetworkAddress`.

### Method 2: Virtual Adapter (Hyper-V)
Creating a software-defined adapter that doesn't exist physically.

1. **Enable Hyper-V:** `Win+R` -> `optionalfeatures` -> Enable "Hyper-V" -> Restart PC.
2. **Create virtual switcher:** Find on your PC "Hyper-V manager" -> Press "Virtual Switch Manager" -> Create new Virtual Switch with "Internal" type.
3. **Set Custom MAC:** Device Manager (`Win+X`) -> Find Adapter -> Properties -> Advanced -> Look for Network Address -> Set the MAC (e.g., `102ABB3C4D5F`).

## 4. Linux Instructions 

### Method 1: Physical Adapter Masking
Directly modifying the settings of your actual network card.

1. **Down the interface:** `sudo ip link set eno1 down`
2. **Set MAC:** `sudo ip link set eno1 address 10:2A:BB:3C:4D:5F`
3. **Up the interface:** `sudo ip link set eno1 up`
4. **Verify:** `ip link show eno1`

### Method 2: Virtual Interface (For Testing and emulation)
Creating an interface that exists only in the kernel, without touching real NICs.

1. **Create:** `sudo ip link add test-vnet type dummy`
2. **Set MAC:** `sudo ip link set test-vnet address 10:2A:BB:3C:4D:5F`
3. **Up:** `sudo ip link set test-vnet up`
4. **Verify:** `ip link show test-vnet`

## 5. Troubleshooting

* **"Invalid Host" Error:**
The software is scanning the wrong interface. Ensure your primary physical interface has the required MAC address and all other interfaces are down.

* **Conflict of Interfaces:**
Disable all unnecessary network interfaces (Wi-Fi, secondary LAN) to ensure the some app manager binds only to your target MAC.