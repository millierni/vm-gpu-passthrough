# Virtual Machine - GPU passthrough
## VIRTUAL MACHINE - GPU PASSTHROUGH [Same Cards/pci-ids]

- Create `vfio-pci-override.sh`
  ```
  sudo nano /usr/local/bin/vfio-pci-override.sh
  ```
- Add this into `vfio-pci-override.sh`
  - `lspci -vnn` to find the GPU PCI buses and modify the DEVS array
  ```
  {

  #!/bin/sh

  DEVS="0000:12:00.0 0000:12:00.1 0000:12:00.2 0000:12:00.3"

  if [ ! -z "$(ls -A /sys/class/iommu)" ]; then
      for DEV in $DEVS; do
          echo "vfio-pci" > /sys/bus/pci/devices/$DEV/driver_override
      done
  fi

  modprobe -i vfio-pci

  }
  ```
- Change the permission and the ownership of `vfio-pci-override.sh`
  ```
  sudo chmod 755 /usr/local/bin/vfio-pci-override.sh
  sudo chown root:root /usr/local/bin/vfio-pci-override.sh
  ```
- Modify `/etc/mkinitcpio.conf`
  ```
  sudo nano /etc/mkinitcpio.conf
  ```
  - Add this to MODULES array
    ```
    vfio_pci vfio vfio_iommu_type1 vfio_virqfd
    ```
  - Add this to HOOKS array
    ```
    modconf
    ```
  - Add this to FILES array
    ```
    /usr/local/bin/vfio-pci-override.sh
    ```
- Create `vfio.conf`
  ```
  sudo nano /etc/modprobe.d/vfio.conf
  ```
  - Add this into `vfio.conf`
    ```
    install vfio-pci /usr/local/bin/vfio-pci-override.sh
    ```
- Make initramfs
  ```
  sudo mkinitcpio -P
  ```
- Reboot
  ```
  sudo reboot
  ```
- Verify that the “Kernel driver in use” is vfio-pci
  ```
  lspci -vnn
  ```
