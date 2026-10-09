# TP Final Robótica Móvil: Robot Omnidireccional

TP final de Robótica Móvil. Un robot con 4 ruedas Mecanum, simulado en CoppeliaSim y controlado con ROS 2 Humble: modelo cinemático y odometría, seguimiento de trayectoria a lazo cerrado, y localización con un EKF a partir de postes detectados con un LiDAR.

## Estructura

| Carpeta | Contenido |
|---|---|
| `paquetes/` | Paquetes ROS 2, uno por módulo del TP (ver abajo) |
| `simulacion/` | Escena de CoppeliaSim (`omni_ekf.ttt`), config de RViz (`tpfinal.rviz`) y URDF del robot (`urdf/`) |
| `docker/` | Imagen con ROS 2 Humble + CoppeliaSim y scripts para levantar el contenedor |
| `scripts/` | Automatización de experimentos: crear, correr, grabar y plotear corridas |
| `experiments/` | Corridas registradas (manifest, notas y gráficos), ver `experiments/README.md` |
| `docs/` | Diseño y decisiones de cada módulo, y guía del informe (`report.md`) |
| `informe/` | Informe final en LaTeX |
| `referencias/` | Consigna, paper del modelo cinemático Mecanum y guía de Docker |

## Módulos

| Módulo | Paquete | Qué hace | Doc |
|---|---|---|---|
| 1. Modelo cinemático | `modelo_holonomico` | Cinemática inversa: `/robot/cmd_vel` → velocidad de cada rueda. Cinemática directa: ticks de encoders → odometría (`/robot/odometry`, TF `odom → base_link`) | `docs/01_modelo_holonomico.md` |
| 2. Lazo cerrado | `lazo_cerrado` | Genera una trayectoria cuadrada de 2 m y la sigue con un controlador P por grado de libertad y selección de objetivo tipo *pursuit* | `docs/02_lazo_cerrado.md` |
| 3. Localización EKF | `ekf_localizer` | `landmark_detector` agrupa el scan del LiDAR en postes (range, bearing); `ekf_localizer` predice con la odometría, corrige con los postes y publica `map → base_link_ekf` | `docs/03_*.md`, `docs/modulo3_*.md` |
| — | `robmovil_msgs` | Mensajes de la materia: encoders, landmarks, trayectorias | — |

El módulo 3 se monta sobre los anteriores: `seguimiento_ekf.launch.py` levanta odometría + EKF + controlador, y el controlador usa la pose del EKF en lugar de la odometría.

## Uso

```bash
# Host: construir la imagen y abrir el contenedor (monta este repo en /root/ros2_ws/src/robotica)
bash docker/start-docker.sh build
bash docker/start-docker.sh start
bash docker/start-docker.sh open
```

La primera vez hay que registrar los mensajes en el plugin de CoppeliaSim (ver `docker/pasos_docker.txt`). Después, dentro del contenedor:

```bash
cd /root/ros2_ws && colcon build --symlink-install && source install/setup.bash

ros2 launch modelo_holonomico odometry.launch.py        # módulo 1
ros2 launch lazo_cerrado lazo_cerrado.launch.py         # módulo 2 (kp_xy:=…, kp_yaw:=…)
ros2 launch ekf_localizer seguimiento_ekf.launch.py     # módulos 1 + 2 + 3
```

Con la escena `simulacion/omni_ekf.ttt` corriendo en CoppeliaSim y RViz abierto con `simulacion/tpfinal.rviz`.

**Experimentos:** `bash scripts/new_experiment.sh <slug> "<descripción>"`, completar el `manifest.yaml`, después `bash scripts/run_experiment.sh <dir>` (dentro del contenedor) y `python3 scripts/plot_run.py <dir>`.

**Test:** `python3 paquetes/ekf_localizer/test/check_jacobians.py` verifica los jacobianos del EKF contra diferencias finitas.
