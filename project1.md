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
<img width="652" height="385" alt="image" src="https://github.com/user-attachments/assets/c0237dbe-40e0-4472-8fb3-518ec3779be8" />

## breakdown
<img width="450" height="304" alt="image" src="https://github.com/user-attachments/assets/338c6fc9-60c0-43b5-b45e-6c6675cd3ec4" />

## msc
<img width="840" height="742" alt="image" src="https://github.com/user-attachments/assets/79b6ae79-6e1b-4aea-9dfe-776740e5e171" />

## FPGA板
https://github.com/user-attachments/assets/c2c6a928-ef4d-4e71-916f-822c4e488533


