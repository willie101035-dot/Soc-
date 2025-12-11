```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;


entity updncounter_v2 is
	generic(
		upper : integer := 9 ; 
		lower : integer := 0
		);
	port(
		clk,rst : in  STD_LOGIC;
		dir : in STD_LOGIC;
		Q : out STD_LOGIC_VECTOR(3 downto 0);
		P : out STD_LOGIC_VECTOR(3 downto 0)
		);
end updncounter_v2;
	
architecture Behavioral of updncounter_v2 is
	signal Qn : integer := lower;
	signal Pn : integer := lower;
begin
	counter1 : process(clk, rst)
		begin
			if rst= '1' then
				Qn <= lower;
			elsif rising_edge(clk) then
				if dir= '1' then
					if Qn >= upper then
						Qn <= lower;
					else
						Qn <= Qn + 1;
					end if;
				else
					if Qn <= lower then
						Qn <= upper;
					else 
						Qn <= Qn - 1;
					end if;
				end if;
			end if;
	end process;
	counter2 : process(clk, rst)
		begin
			if rst= '1' then
				Pn <= upper;
			elsif rising_edge(clk) then
				if dir= '1' then
					if Pn <= lower then
						Pn <= upper;
					else
						Pn <= Pn - 1;
					end if;
				else
					if Pn >= upper then
						Pn <= lower;
					else 
						Pn <= Pn + 1;
					end if;
				end if;
			end if;
	end process;
	Q <= std_logic_vector(to_unsigned(Qn, Q'length));
	P <= std_logic_vector(to_unsigned(Pn, P'length));
end Behavioral;
```
