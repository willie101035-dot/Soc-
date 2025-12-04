# up_counter & downc_counter

## 程式碼
''' vhdl
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
''' vhdl
