# GROMACS 分子动力学模拟 小分子自组装

## 目录

- [1. 从 PubChem 下载分子结构](#1-从-pubchem-下载分子结构)
- [2. 将 `.sdf` 文件转换为 `mol2` 文件](#2-将-sdf-文件转换为-mol2-文件)
- [3. 使用 ORCA 在 B97-3c 级别下优化分子](#3-使用-orca-在-b97-3c-级别下优化分子)
- [4. 将 ORCA 输出文件转换为 `.pdb` 文件](#4-将-orca-输出文件转换为-pdb-文件)
- [5. 生成拓扑文件（`.top`）](#5-生成拓扑文件top)
  - [方法一：GMXTOP](#方法一gmxtop)
  - [方法二：acpype.py（本地脚本）](#方法二acpypepy本地脚本)
  - [方法三：sobtop.exe（推荐用于 amber99sb.ff）](#方法三sobtopexe推荐用于-amber99sbff)
- [6. 创建并编辑 `.itp` 文件](#6-创建并编辑-itp-文件)
- [7. 构建初始构型](#7-构建初始构型)
  - [推荐使用 Packmol](#推荐使用-packmol)
  - [或使用 GROMACS 的 `insert-molecules`](#或使用-gromacs-的-insert-molecules)
- [8. 运行 GROMACS 模拟](#8-运行-gromacs-模拟)
  - [8.1 溶剂化](#81-溶剂化)
  - [8.2 添加离子（可选）](#82-添加离子可选)
  - [8.3 能量最小化（EM）](#83-能量最小化em)
  - [8.4 NVT 平衡（模拟退火）](#84-nvt-平衡模拟退火)
  - [8.5 NPT 平衡](#85-npt-平衡)
  - [8.6 生产动力学（MD）](#86-生产动力学md)
  - [8.7 轨迹后处理（Trjconv）](#87-轨迹后处理trjconv)
  - [8.8 常用分析](#88-常用分析)
    - [距离计算（`gmx distance`）](#距离计算gmx-distance)
    - [RMSD（`gmx rms`）](#rmsdgmx-rms)
    - [径向分布函数 RDF（`gmx rdf`）](#径向分布函数-rdfgmx-rdf)
    - [回旋半径（`gmx gyrate`）](#回旋半径gmx-gyrate)
    - [溶剂可及表面积 SASA（`gmx sasa`）](#溶剂可及表面积-sasagmx-sasa)
    - [均方位移 MSD（`gmx msd`）](#均方位移-msdgmx-msd)
    - [结合自由能（MM/PBSA）](#结合自由能mmpbsa)

---

## 1. 从 PubChem 下载分子结构

## 2. 将 `.sdf` 文件转换为 `mol2` 文件

可使用 Gaussian、Avogadro 等软件完成。  
**提示**：也可直接用 Gaussian 或 Avogadro 构建结构，或在 Avogadro 中通过 `Build → Insert → SMILES` 导入 SMILES 字符串。

## 3. 使用 ORCA 在 B97-3c 级别下优化分子（`.sdf` 文件）

建议使用 Multiwfn 生成 ORCA 输入文件，示例头部：
```
maxcore 5000
pal nprocs 16 end
```

## 4. 将 ORCA 输出文件（`.out`）转换为 `.pdb` 文件

使用 Avogadro 进行转换。

## 5. 生成拓扑文件（`.top`）

### 方法一：GMXTOP  
访问 [https://jerkwin.github.io/prog/gmxtop.html](https://jerkwin.github.io/prog/gmxtop.html)

### 方法二：acpype.py（本地脚本）  
脚本路径：`/home/yang/Documents/lyon/MD231106/`  
**注意**：使用前需将脚本第 3185 行的 `"mopac"` 改为 `"sqm"`。  
```bash
./acpype.py -i molecule.mol2
```
运行后会生成一个文件夹，包含 `.gro`、`.itp` 和 `.top` 文件。可能需要手动编写位置限制文件（`porse.top`）。

### 方法三：sobtop.exe（推荐用于 amber99sb.ff）

## 6. 创建并编辑 `.itp` 文件

## 7. 构建初始构型

### 推荐使用 Packmol
```bash
Packmol < input.inp
```

### 或使用 GROMACS 的 `insert-molecules`
```bash
gmx insert-molecules -ci 14982.pdb -nmol 63 -box 12 12 12 -o box_14982.gro
```

**注意**：
- 记得编辑 `topol.top` 文件。
- 编辑 `.pdb` 和 `.itp` 文件，将 `MOL` 替换为实际的分子名称。

## 8. 运行 GROMACS 模拟

### 8.1 溶剂化
```bash
gmx solvate -cp "box".gro -cs spc216.gro -o solv.gro -p topol.top
```

### 8.2 添加离子（可选）
```bash
gmx grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr
echo 13 | gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral
# 选择溶剂组（Select sol）
```

### 8.3 能量最小化（EM）
```bash
gmx grompp -f em.mdp -c solv_ions.gro -p topol.top -o em.tpr
gmx mdrun -deffnm em -v
```

### 8.4 NVT 平衡（模拟退火）
```bash
gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr
gmx mdrun -deffnm nvt -v
```

### 8.5 NPT 平衡
```bash
gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -p topol.top -o npt.tpr
gmx mdrun -deffnm npt -v
```

### 8.6 生产动力学（MD）
```bash
gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr
gmx mdrun -deffnm md_0_1 -v -ntmpi 6 -ntomp 8
```

**提示**：使用 `chmod +x ./test.sh` 开启脚本执行权限。

### 8.7 轨迹后处理（Trjconv）
```bash
# 创建索引文件
gmx make_ndx -f npt.gro -o index.ndx

# 去除周期性边界条件（PBC）
gmx trjconv -s md_0_1.tpr -f md_0_1.xtc -o md_0_1_noPBC.xtc -pbc cluster -center -n index.ndx

# 提取最后一帧（例如 200 ns 处）
gmx trjconv -s md_0_1.tpr -f md_0_1_noPBC.xtc -o end.pdb -dump 200000 -n index.ndx
```

### 8.8 常用分析

#### 距离计算（`gmx distance`）
计算两个位置之间的距离。

#### RMSD（`gmx rms`）
```bash
gmx rms -s md_0_1.tpr -f md_0_1_noPBC.xtc -n index.ndx -o rmsd.xvg -b 0 -e 50 -dt 0.1 -tu ns
```

#### 径向分布函数 RDF（`gmx rdf`）
```bash
gmx rdf -s md_0_1.tpr -f md_0_1_noPBC.xtc -n index.ndx -o rdf.xvg -b 0 -e 50 -dt 0.1
gmx rdf -s md_0_1.tpr -f md_0_1_noPBC.xtc -n index.ndx -o rdf.xvg -b 0 -e 200 -dt 0.1 -cn
```

#### 回旋半径（`gmx gyrate`）
```bash
gmx gyrate -s md_0_1.tpr -f md_0_1_noPBC.xtc -n index.ndx -o gyrate.xvg
```

#### 溶剂可及表面积 SASA（`gmx sasa`）
```bash
gmx sasa -f md_0_1_noPBC.xtc -s md_0_1.tpr -n index.ndx -tu ns -odg -surface -o area.xvg -b 0 -e 50 -dt 0.1
```

#### 均方位移 MSD（`gmx msd`）
```bash
gmx msd -f md_0_1_noPBC.xtc -s md_0_1.tpr -n index.ndx -o msd.xvg -b 0 -e 100 -tu ns -beginfit 20 -endfit 100
```

#### 结合自由能（MM/PBSA）

**真空势能**  
```bash
g_mmpbsa -f trj_200ns.xtc -s md_0_1.tpr -n index.ndx -pdie 2 -decomp
```

**极性溶剂化能**  
```bash
g_mmpbsa -f md_0_1_noPBC.xtc -s md_0_1.tpr -n index.ndx -i polar.mdp -nomme -pbsa -decomp
```

**非极性溶剂化能**  
```bash
g_mmpbsa -f md_0_1_noPBC.xtc -s md_0_1.tpr -n index.ndx -i apolar_sasa.mdp -nomme -pbsa -decomp -apol sasa.xvg -apcon sasa_contrib.dat
```

**平均结合自由能**  
```bash
python3 MmPbSaStat.py -m energy_MM.xvg -p polar.xvg -a sasa.xvg
```
