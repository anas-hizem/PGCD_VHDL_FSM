
# Calcul du PGCD en VHDL avec Machine à États Finis (FSM)

## Description

Ce laboratoire présente une implémentation du calcul du plus grand diviseur commun (PGCD) de deux nombres entiers en VHDL, en utilisant une machine à états finis (FSM). Le PGCD est calculé par soustraction répétée entre les deux entiers jusqu'à ce qu'ils soient égaux. Ce processus est décrit à travers une FSM avec six états distincts.

### Fonctionnalités :

- **Entrées** :
  - `ina`, `inb` : Les deux entiers pour lesquels le PGCD doit être calculé.
  - `start` : Un signal de démarrage pour initier le calcul.
  - `clock`, `reset` : Les signaux d'horloge et de réinitialisation.

- **Sorties** :
  - `o` : Le PGCD calculé des deux entiers.
  - `done` : Un signal indiquant la fin du calcul.

### Machine à États Finis (FSM) :

La FSM utilisée pour implémenter le calcul du PGCD est composée des états suivants :
1. **INIT** : Initialisation.
2. **READIN** : Lecture des entrées.
3. **COMPARE** : Comparaison des deux entiers.
4. **DECA** : Soustraction de `a` par `b` si `a > b`.
5. **DECB** : Soustraction de `b` par `a` si `b > a`.
6. **FINISH** : Le PGCD a été trouvé et le calcul est terminé.

## Code VHDL

### Entité `gcd` (Calcul du PGCD)

```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
USE ieee.numeric_std.ALL;

ENTITY gcd IS
    PORT (
        ina, inb : IN STD_LOGIC_VECTOR(3 DOWNTO 0);
        Start, clock, reset : IN STD_LOGIC;
        o : OUT STD_LOGIC_VECTOR(3 DOWNTO 0);
        done : OUT STD_LOGIC
    );
END gcd;

ARCHITECTURE Behavioral OF gcd IS
    TYPE states IS (INIT, READIN, COMPARE, DECA, DECB, FINISH);
    SIGNAL currentstate, nextstate : states;
    SIGNAL a, a_in : unsigned(3 DOWNTO 0);
    SIGNAL b, b_in : unsigned(3 DOWNTO 0);

BEGIN

    PROCESS (clock, reset)
    BEGIN
        IF (reset = '0') THEN
            currentstate <= INIT;
            a <= (OTHERS => '0');
            b <= (OTHERS => '0');
        ELSIF (clock'event AND clock = '1') THEN
            currentstate <= nextstate;
            a <= a_in;
            b <= b_in;
        END IF;
    END PROCESS;

    PROCESS (currentstate, start, a, b)
    BEGIN
        CASE currentstate IS
            WHEN INIT =>
                IF (start = '0') THEN
                    nextstate <= INIT;
                ELSE
                    nextstate <= READIN;
                END IF;

            WHEN READIN =>
                nextstate <= COMPARE;

            WHEN COMPARE =>
                IF (a = b) THEN
                    nextstate <= FINISH;
                ELSIF (a > b) THEN
                    nextstate <= DECA;
                ELSE
                    nextstate <= DECB;
                END IF;

            WHEN DECA =>
                nextstate <= COMPARE;

            WHEN DECB =>
                nextstate <= COMPARE;

            WHEN FINISH =>
                IF (start = '0') THEN
                    nextstate <= INIT;
                ELSE
                    nextstate <= FINISH;
                END IF;

            WHEN OTHERS =>
                nextstate <= INIT;
        END CASE;
    END PROCESS;

    PROCESS (currentstate, ina, a, b)
    BEGIN
        CASE currentstate IS
            WHEN READIN =>
                a_in <= unsigned(ina);
            WHEN DECA =>
                a_in <= a - b;
            WHEN OTHERS =>
                a_in <= a;
        END CASE;
    END PROCESS;

    PROCESS (currentstate, ina, a, b)
    BEGIN
        CASE currentstate IS
            WHEN READIN =>
                b_in <= unsigned(inb);
            WHEN DECB =>
                b_in <= b - a;
            WHEN OTHERS =>
                b_in <= b;
        END CASE;
    END PROCESS;

    o <= STD_LOGIC_VECTOR(a);

    PROCESS (currentstate)
    BEGIN
        CASE currentstate IS
            WHEN FINISH =>
                done <= '1';
            WHEN OTHERS =>
                done <= '0';
        END CASE;
    END PROCESS;

END ARCHITECTURE;
```

### Banc de Test (`gcd_tb`)

```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
USE ieee.numeric_std.ALL;

ENTITY gcd_tb IS
END ENTITY;

ARCHITECTURE tb OF gcd_tb IS

  COMPONENT gcd IS
    PORT (
      ina, inb : IN STD_LOGIC_VECTOR(3 DOWNTO 0);
      Start, clock, reset : IN STD_LOGIC;
      o : OUT STD_LOGIC_VECTOR(3 DOWNTO 0);
      done : OUT STD_LOGIC
    );
  END COMPONENT;

  SIGNAL rst_tb : STD_LOGIC;
  SIGNAL clk_tb : STD_LOGIC := '0';
  SIGNAL start_tb : STD_LOGIC := '1';
  SIGNAL ina_tb : STD_LOGIC_VECTOR(3 DOWNTO 0) := "1110";
  SIGNAL inb_tb : STD_LOGIC_VECTOR(3 DOWNTO 0) := "0110";
  SIGNAL dout_tb : STD_LOGIC_VECTOR(3 DOWNTO 0);
  SIGNAL done_tb : STD_LOGIC;

  CONSTANT hp : TIME := 50 ns;

BEGIN

  DUT : gcd
  PORT MAP(
    ina => ina_tb,
    inb => inb_tb,
    reset => rst_tb,
    clock => clk_tb,
    start => start_tb,
    o => dout_tb,
    done => done_tb
  );

  clk_tb <= NOT clk_tb AFTER hp;
  rst_tb <= '0', '1' AFTER 2 * hp;

END tb;
```

## Testbench

Le testbench pour ce projet simule les valeurs de `ina` et `inb` et valide les sorties. L'horloge est simulée avec une période de 50 ns et un reset initial est appliqué.

- **Entrées Testées** :
  - `ina = 1110` (14 en décimal)
  - `inb = 0110` (6 en décimal)
  - Le signal `start` est activé pour démarrer le calcul.

## Chronogramme

Le chronogramme montre l'évolution des signaux au cours de la simulation. On peut observer les transitions entre les différents états de la FSM (INIT, READIN, COMPARE, DECA, DECB, FINISH) et les changements dans les valeurs de `a`, `b`, et la sortie `o`.

Voici une capture d'écran typique du chronogramme (généré à partir de ModelSim) :

![Chronogramme](https://github.com/user-attachments/assets/2711067c-b4a2-4898-9039-dd78508e0134)


## Netlist

La netlist générée par Quartus Prime Lite pour l'implémentation sur un FPGA montre les connexions physiques entre les composants logiques. Vous pouvez générer la netlist en utilisant Quartus Prime Lite après avoir compilé le projet VHDL.

![Netlist](https://github.com/user-attachments/assets/4a90b01a-fdb6-4212-b436-51ae67d04421)


## Outils Utilisés

- **Quartus Prime Lite** : Logiciel pour la conception et l'implémentation sur FPGA.
- **ModelSim** : Outil de simulation pour vérifier le bon fonctionnement du code VHDL avant la synthèse.

## Conclusion

Ce projet montre comment implémenter un calcul de PGCD en utilisant VHDL avec une FSM. Il inclut des étapes de simulation dans ModelSim et une implémentation matérielle sur un FPGA avec Quartus Prime Lite.
