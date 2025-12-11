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
<img width="839" height="539" alt="Image" src="https://github.com/user-attachments/assets/730dcad1-08e2-4b48-9b2e-581f85b65be4" />
<img width="839" height="539" alt="Image" src="https://github.com/user-attachments/assets/730dcad1-08e2-4b48-9b2e-581f85b65be4" />
<img width="1001" height="747" alt="Image" src="https://github.com/user-attachments/assets/1528f832-efc1-4189-8c53-c2b413ad2fec" />
<img width="1118" height="509" alt="Image" src="https://github.com/user-attachments/assets/1ee5e2fa-b420-40fb-b832-a5857d64a24d" />
<img width="956" height="452" alt="Image" src="https://github.com/user-attachments/assets/ede971d7-1ba9-43ef-839e-7d0196a7c73f" />
