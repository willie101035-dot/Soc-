#FSM_counter

## 程式碼
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity FSM_counter is
    generic(
        upper : integer := 9;
        lower : integer := 0
    );
    port(
        rst : in  STD_LOGIC;
        clk : in  STD_LOGIC;
        q   : out STD_LOGIC_VECTOR(3 downto 0)
    );
end FSM_counter;

architecture Behavioral of FSM_counter is

    type counter_state is (counter1, counter2);  
    signal now_state, next_state : counter_state;
    signal Qn, Qn_next           : integer := 0;

begin


    process (rst, clk)
    begin
        if rst = '1' then
            now_state <= counter1;  
            Qn        <= lower;     
        elsif rising_edge(clk) then
            now_state <= next_state;
            Qn        <= Qn_next;
        end if;
    end process;


    process (now_state, Qn)
    begin
     
        next_state <= now_state;
        Qn_next    <= Qn;

        case now_state is


            when counter1 =>
                if Qn >= upper then     
                    next_state <= counter2;
                    Qn_next    <= Qn - 1; 
                else
                    Qn_next    <= Qn + 1;
                end if;

          
            when counter2 =>
                if Qn <= lower then     
                    next_state <= counter1;
                    Qn_next    <= Qn + 1;
                else
                    Qn_next    <= Qn - 1; 
                end if;

        end case;
    end process;

    q <= std_logic_vector(to_unsigned(Qn, 4));

end Behavioral;
```
## 波形圖
<img width="922" height="578" alt="image" src="https://github.com/user-attachments/assets/27c58ced-f860-4d5d-9803-c4aec2f2cd46" />

## breakdown
<img width="481" height="356" alt="image" src="https://github.com/user-attachments/assets/2aa178f1-8c86-48f4-8dad-0731d5038b4e" />
