# up_counter & downc_counter

## 程式碼
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity updncounter_vhdl is
port(
rst : in STD_LOGIC;
clk : in STD_LOGIC;
q : out STD_LOGIC_vector(3 downto 0);
p : out STD_LOGIC_vector(3 downto 0)
);
end updncounter_vhdl;

architecture rtl1 of updncounter_vhdl is
signal Qn : STD_LOGIC_vector(3 downto 0) := (others => '0');
signal Pn : STD_LOGIC_vector(3 downto 0) := "1001";
begin

process(clk, rst)
begin
    if rst = '1' then
        Qn <= (others => '0'); 
    elsif rising_edge(clk) then
        if unsigned(Qn) < 9 then
            Qn <= std_logic_vector(unsigned(Qn) + 1);
        else
            Qn <= Qn;           
        end if;
    end if;
end process;


process(clk, rst)
begin
    if rst = '1' then
        Pn <= "1001";           
    elsif rising_edge(clk) then
        if unsigned(Pn) > 0 then
            Pn <= std_logic_vector(unsigned(Pn) - 1);
        else
            Pn <= Pn;          
        end if;
    end if;
end process;

q <= Qn;
p <= Pn;
end rtl1;

```
## 波形圖
<img width="912" height="499" alt="image" src="https://github.com/user-attachments/assets/2e44dc64-d465-4630-9213-3fbf527aa777" />

## 架構圖
<img width="543" height="282" alt="image" src="https://github.com/user-attachments/assets/e1738035-828b-40e4-b7cb-d7e3e2b41b65" />

## AOV
<img width="630" height="397" alt="image" src="https://github.com/user-attachments/assets/1fc39c1a-a596-44dc-a3ec-d3c890008b3a" />

## breakdown
<img width="498" height="330" alt="image" src="https://github.com/user-attachments/assets/1eb9cb93-e548-4f57-af59-7cc5d5e4ffbf" />

## msc
<img width="1168" height="316" alt="image" src="https://github.com/user-attachments/assets/b3084091-9156-4685-bf68-c9a89a2c6a4f" />


## FPGA板
https://github.com/user-attachments/assets/c2c6a928-ef4d-4e71-916f-822c4e488533

