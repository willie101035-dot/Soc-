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

    signal Qn, Qn_d : integer := lower;
begin

    process(clk, rst)
    begin
        if rst = '1' then
            now_state <= counter1;
        elsif rising_edge(clk) then
            now_state <= next_state;
        end if;
    end process;

    process(now_state, Qn)
    begin
        next_state <= now_state;

        case now_state is
            when counter1 =>
                if Qn >= upper then
                    next_state <= counter2;
                end if;

            when counter2 =>
                if Qn <= lower then
                    next_state <= counter1;
                end if;
        end case;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            Qn <= lower;
        elsif rising_edge(clk) then
            Qn <= Qn_d;
        end if;
    end process;

    process(now_state, Qn)
    begin
        Qn_d <= Qn;

        case now_state is
            when counter1 =>
                if Qn >= upper then
                    Qn_d <= Qn - 1;
                else
                    Qn_d <= Qn + 1;
                end if;

            when counter2 =>
                if Qn <= lower then
                    Qn_d <= Qn + 1;
                else
                    Qn_d <= Qn - 1;
                end if;
        end case;
    end process;

    q <= std_logic_vector(to_unsigned(Qn, 4));

end Behavioral;

```
## 波形圖
<img width="840" height="506" alt="image" src="https://github.com/user-attachments/assets/18026499-3831-4989-9a34-c76fbebe3d3a" />

## breakdown
<img width="1196" height="528" alt="image" src="https://github.com/user-attachments/assets/a423cfae-7fb3-456c-83a2-56a5ed3a1a85" />

## AOV
<img width="734" height="227" alt="image" src="https://github.com/user-attachments/assets/407a6ac9-d609-4e3f-833b-c5182c6483ca" />

## 架構圖
<img width="525" height="487" alt="image" src="https://github.com/user-attachments/assets/a36a8597-5bbc-4e69-98d1-c442b020f76e" />

## MSC
<img width="1103" height="316" alt="image" src="https://github.com/user-attachments/assets/894c5731-59ed-4dab-82f2-9e9d5b89519a" />

## FSM
<img width="1400" height="801" alt="image" src="https://github.com/user-attachments/assets/db13b9c6-2f04-4c11-adc8-4e6e2289b087" />

## fpga板
https://github.com/user-attachments/assets/785d42d4-4750-48c5-975f-4dc707541a71

